---
title: "2018-01-23-when-signal-handler-is-executed"
date: 2018-01-23 15:24
---

###when signal handler is executed

I'm always confusion that signal is like a interrupt, but it's can't be a real interrupt. after google and study the kernel code. the conclusion is when a process switch from kernel mode to user mode.

###study of the kernel code

the kernel version is 4.14

the code of dealing a syscall is in arch/x86/entry/common.c

```C
__visible void do_syscall_64(struct pt_regs *regs)
{
	struct thread_info *ti = current_thread_info();
	unsigned long nr = regs->orig_ax;

	enter_from_user_mode();
	local_irq_enable();

	if (READ_ONCE(ti->flags) & _TIF_WORK_SYSCALL_ENTRY)
		nr = syscall_trace_enter(regs);

	/*
	 * NB: Native and x32 syscalls are dispatched from the same
	 * table.  The only functional difference is the x32 bit in
	 * regs->orig_ax, which changes the behavior of some syscalls.
	 */
	if (likely((nr & __SYSCALL_MASK) < NR_syscalls)) {
		regs->ax = sys_call_table[nr & __SYSCALL_MASK](
			regs->di, regs->si, regs->dx,
			regs->r10, regs->r8, regs->r9);
	}

	syscall_return_slowpath(regs);
}
```

the `do_syscall_64` call `syscall_return_slowpath` to return from a syscall:

```C
__visible inline void syscall_return_slowpath(struct pt_regs *regs)
{
	struct thread_info *ti = current_thread_info();
	u32 cached_flags = READ_ONCE(ti->flags);

	CT_WARN_ON(ct_state() != CONTEXT_KERNEL);

	if (IS_ENABLED(CONFIG_PROVE_LOCKING) &&
	    WARN(irqs_disabled(), "syscall %ld left IRQs disabled", regs->orig_ax))
		local_irq_enable();

	/*
	 * First do one-time work.  If these work items are enabled, we
	 * want to run them exactly once per syscall exit with IRQs on.
	 */
	if (unlikely(cached_flags & SYSCALL_EXIT_WORK_FLAGS))
		syscall_slow_exit_work(regs, cached_flags);

	local_irq_disable();
	prepare_exit_to_usermode(regs);
}
```

the `syscall_return_slowpath` call `prepare_exit_to_usermode` to check if there is extra work to if there is call `exit_to_usermode_loop` to do the work, and the pending signals is handled here.

```C
static void exit_to_usermode_loop(struct pt_regs *regs, u32 cached_flags)
{
	/*
	 * In order to return to user mode, we need to have IRQs off with
	 * none of EXIT_TO_USERMODE_LOOP_FLAGS set.  Several of these flags
	 * can be set at any time on preemptable kernels if we have IRQs on,
	 * so we need to loop.  Disabling preemption wouldn't help: doing the
	 * work to clear some of the flags can sleep.
	 */
	while (true) {
		/* We have work to do. */
		local_irq_enable();

		if (cached_flags & _TIF_NEED_RESCHED)
			schedule();

		if (cached_flags & _TIF_UPROBE)
			uprobe_notify_resume(regs);

		/* deal with pending signal delivery */
		if (cached_flags & _TIF_SIGPENDING)
			do_signal(regs);

		if (cached_flags & _TIF_NOTIFY_RESUME) {
			clear_thread_flag(TIF_NOTIFY_RESUME);
			tracehook_notify_resume(regs);
		}

		if (cached_flags & _TIF_USER_RETURN_NOTIFY)
			fire_user_return_notifiers();

		if (cached_flags & _TIF_PATCH_PENDING)
			klp_update_patch_state(current);

		/* Disable IRQs and retry */
		local_irq_disable();

		cached_flags = READ_ONCE(current_thread_info()->flags);

		if (!(cached_flags & EXIT_TO_USERMODE_LOOP_FLAGS))
			break;
	}
}
```