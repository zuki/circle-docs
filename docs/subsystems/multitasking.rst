.. _Multitasking:

マルチタスク
~~~~~~~~~~~~

Circleはオプションで協調型のノンプリエンプティブスケジューラを提供しており、プロセスコンセプトに基づいたプログラミングの問題を解くことができます。Circleでは物理アドレスと仮想アドレスの1対1のマッピングを持つフラットなアドレス空間しかないため、Circleのプロセスはスレッドに似ています。Circleでは代わりに「タスク"」という名前が使用されています。

スケジューラはオプションなので、Circleアプリケーションはスケジューラ無しでも動作します。スケジューラはTCP/IPネットワーキングのサポートを実装するために導入されました。そこでは、1コアモデルのRaspberry Piでも同時に多くのスレッドを実行する必要があったからです。その後、VCHIQドライバとHDMIサウンドのサポート、アクセラレーテッドグラフィックスのサポート（Raspberry Pi 1-3とZeroのみ）、無線LANのサポートが移植されましたが、同様の要件がありました。Circleで複雑なユーザ問題をモデル化する際にもスケジューラを使うと便利です。

スケジューラライブラリは以下のクラスを提供します。

======================  ===================================================
クラス                  機能
======================  ===================================================
CScheduler              協調型非プリエンプティブスケジューラ
CTask                   実行スレッドの表現、タスク
CMutex                  タスク間の相互排除（クリティカルセクション）
CSemaphore              よく知られたセマフォ概念の実装
CSynchronizationEvent   イベントを使ってタスクの実行を同期
======================	===================================================

.. important::

	マルチコア環境 (:ref:`Multi-core support` を参照) ではすべてのタスクとスケジューラはCPUコア0で実行されます。

CScheduler
^^^^^^^^^^

このクラスは協調型のノンプリエンプティブスケジューラを実装しており、ある時点でどのタスクを実行するかを制御します。このスケジューラはノンプリエンプティブなので、他のタスクが実行できるように、実行中のタスクはスリープするか、同期オブジェクト（ミューテックス、セマフォ、同期イベント）を待つか、しばらくしてから ``CScheduler::Get()->Yield()`` を呼び出すことによって明示的に CPU を解放する必要があります。この比較的単純なスケジューラはタスク優先順位のない（そしてそれほどオーバーヘッドのない）ラウンドロビン方式を実装しています。

.. code-block:: cpp

	#include <circle/sched/scheduler.h>

.. cpp:class:: CScheduler

.. cpp:function:: static boolean CScheduler::IsActive (void)

	システムでスケジューラが利用可能バア愛、 ``TRUE`` を返します。Circleではスケジューラはオプションです。

.. cpp:function:: static CScheduler *CScheduler::Get (void)

	システムでゆういつのスケジューラオブジェクトへのポインタを返します。スケジューラが利用可能でない場合はこのメソッドは呼び出してはいけません。

.. cpp:function:: CTask *CScheduler::GetCurrentTask (void)

	現在実行中のタスクの ``CTask`` オブジェクトへのポインタを返します。

.. cpp:function:: CTask *CScheduler::GetTask (const char *pTaskName)

	名前が ``pTaskName`` の ``CTask`` オブジェクトへのポインタを返します。その名前のタスクが存在しない場合は 0 を返します。

.. cpp:function:: boolean CScheduler::IsValidTask (CTask *pTask)

	Returns ``TRUE``, if ``pTask`` is referencing a CTask object of a currently known task.

.. cpp:function:: void CScheduler::Yield (void)

	Switch to the next task. The currently running task releases the CPU and the next task in round-robin order, which is not blocked, gets control.

.. important::

	A task should call this from time to time, if it does longer calculations.

.. cpp:function:: void CScheduler::Sleep (unsigned nSeconds)

	The current task pauses execution for ``nSeconds`` seconds. The next ready task gets control.

.. cpp:function:: void CScheduler::MsSleep (unsigned nMilliSeconds)

	The current task pauses execution for ``nMilliSeconds`` milliseconds. The next ready task gets control.

.. cpp:function:: void CScheduler::usSleep (unsigned nMicroSeconds)

	The current task pauses execution for ``nMicroSeconds`` microseconds. The next ready task gets control.

.. cpp:function:: void CScheduler::SuspendNewTasks (void)

	Causes all new tasks to be created in a suspended state. You can achieve the same, if you set the parameter ``bCreateSuspended`` to ``TRUE``, when calling ``new`` for a task. Nested calls to ``SuspendNewTasks()`` and ``ResumeNewTasks()`` are allowed.

.. cpp:function:: void CScheduler::ResumeNewTasks (void)

	Stops causing new tasks to be created in a suspended state and starts any tasks that were created suspended. Nested calls to ``SuspendNewTasks()`` and ``ResumeNewTasks()`` are allowed.

.. cpp:function:: void CScheduler::ListTasks (CDevice *pTarget)

	Writes a task listing to the device ``pTarget``.

.. cpp:function:: void CScheduler::RegisterTaskSwitchHandler (TSchedulerTaskHandler *pHandler)

	``pHandler`` is called on each task switch. This method is normally used by the Linux kernel driver and Pthreads emulation. The handler is called with a pointer to the ``CTask`` object of the task, which gets control now. The prototype of the handler is:

.. code-block:: c

	void TSchedulerTaskHandler (CTask *pTask);

.. cpp:function:: void CScheduler::RegisterTaskTerminationHandler (TSchedulerTaskHandler *pHandler)

	``pHandler`` is called, when a task terminates. This method is normally used by the Linux kernel driver and Pthreads emulation. The handler is called with a pointer to the ``CTask`` object of the task, which terminates. See ``RegisterTaskSwitchHandler()`` for the prototype of the handler.

CTask
^^^^^

このクラスを継承して ``Run()`` メソッドを定義して独自のタスクを実装し ``new`` を呼び出して起動します。

.. code-block:: cpp

	#include <circle/sched/task.h>

.. cpp:class:: CTask

.. cpp:function:: CTask::CTask (unsigned nStackSize = TASK_STACK_SIZE, boolean bCreateSuspended = FALSE)

	Creates a task. ``nStackSize`` is the stack size for this task. By default a new task is immediately ready to run and its ``Run()`` method can be called. If you have to do more initialization, before the task can run, set ``bCreateSuspended`` to ``TRUE``. The task has to be started explicitly by calling ``Start()`` on it then.

.. cpp:function:: virtual void CTask::Run (void)

	Override this method to define the entry point for your own task. The task is automatically terminated, when ``Run()`` returns.

.. cpp:function:: void CTask::Start (void)

	Starts a task, that was created with ``bCreateSuspended = TRUE`` or restarts it after ``Suspend()``.

.. cpp:function:: void CTask::Suspend (void)

	Suspends a task from running, until ``Resume()`` is called for this task.

.. cpp:function:: void CTask::Resume (void)

	Alternative method to (re-)start a suspended task.

.. cpp:function:: boolean CTask::IsSuspended (void) const

	Returns ``TRUE``, if the task is currently suspended from running.

.. cpp:function:: void CTask::Terminate (void)

	Terminates the execution of the task. This method can only be called by the task itself. The task terminates on return from ``Run()`` too.

.. cpp:function:: void CTask::WaitForTermination (void)

	Waits for the termination of the task. This method can only be called by an other task.

.. cpp:function:: void CTask::SetName (const char *pName)

	Sets the specific name ``pName`` for this task.

.. cpp:function:: const char *CTask::GetName (void) const

	Returns a pointer to 0-terminated name string of this task. The default name of a task is constructed from the address of its task object (e.g. ``"@84abc0"``). The main application task has the name ``"main"``.

.. cpp:function:: void CTask::SetUserData (void *pData, unsigned nSlot)

	Sets a user pointer for this task. If you have to associate some data with a task, you can call this method with ``nSlot = TASK_USER_DATA_USER``. ``pData`` is any user pointer to be set.

.. cpp:function:: void *CTask::GetUserData (unsigned nSlot)

	Returns a user pointer for this task, which has previously been set using ``SetUserData()``. ``nSlot`` must be ``TASK_USER_DATA_USER`` for application usage.

CMutex
^^^^^^

タスク間の相互排除（クリティカルセクション）を提供する方法を提供します。

.. code-block:: cpp

	#include <circle/sched/mutex.h>

.. cpp:class:: CMutex

.. cpp:function:: void CMutex::Acquire (void)

	Acquires the mutex. The current task blocks, if another task already acquired the mutex. The mutex can be acquired multiple times by the same task.

.. cpp:function:: void CMutex::Release (void)

	Releases the mutex. Another task, which was waiting for the mutex to acquire, will be waken.

CSemaphore
^^^^^^^^^^

ダイクストラによりはじめに定義された `セマフォ <https://en.wikipedia.org/wiki/Semaphore_(programming)>`_ というよく知られてる同期概念を実装しています。このクラスは ``Down()`` 操作でデクリメントする非負のカウンタを保持します。カウンタが既に 0 であり、デクリメントできない場合、呼び出し元のタスクはカウンタが再びインクリメントされるまで待ちます。インクリメントは ``Up()`` 操作でできます。セマフォは限られた数のリソースへのアクセスを制御するために使用できます。

.. code-block:: cpp

	#include <circle/sched/semaphore.h>

.. cpp:class:: CSemaphore

.. cpp:function:: CSemaphore::CSemaphore (unsigned nInitialCount = 1)

	Creates a semaphore. ``nInitialCount`` is the initial count of the semaphore.

.. cpp:function:: unsigned CSemaphore::GetState (void) const

	Returns the current count of the semaphore.

.. cpp:function:: void CSemaphore::Down (void)

	Decrements the semaphore count. Blocks the calling task, if the count is already zero.

.. cpp:function:: void CSemaphore::Up (void)

	Increments the semaphore count. Wakes another waiting task, if the count was zero. Can be called from interrupt context.

.. cpp:function:: boolean CSemaphore::TryDown (void)

	Tries to decrement the semaphore count. Returns ``TRUE`` on success or ``FALSE``, if the count is zero.

CSynchronizationEvent
^^^^^^^^^^^^^^^^^^^^^

タスクの実行をイベントに同期させる方法を提供しまう。イベントはセットとクリアが可能です。タスクがイベントを待っている場合、タスクはイベントがクリア（アンセット）されるとブロックされ、イベントが再びセットされると実行を継続します。複数のタスクが同時にイベントを待つことができます。

.. code-block:: cpp

	#include <circle/sched/synchronizationevent.h>

.. cpp:class:: CSynchronizationEvent

.. cpp:function:: CSynchronizationEvent::CSynchronizationEvent (boolean bState = FALSE)

	Creates the synchronization event. ``bState`` is the initial state of the event (default cleared).

.. cpp:function:: boolean CSynchronizationEvent::GetState (void)

	Returns the current state for the synchronization event.

.. cpp:function:: void CSynchronizationEvent::Clear (void)

	Clears the synchronization event.

.. cpp:function:: void CSynchronizationEvent::Set (void)

	Sets the synchronization event.  Wakes all tasks currently waiting for the event. Can be called from interrupt context.

.. cpp:function:: void CSynchronizationEvent::Wait (void)

	Blocks the calling task, if the synchronization event is cleared. The task will wake up, when the event is set later. Multiple tasks can wait for the event to be set.

.. cpp:function:: boolean CSynchronizationEvent::WaitWithTimeout (unsigned nMicroSeconds)

	Blocks the calling task for ``nMicroSeconds`` microseconds, if the synchronization event is cleared. The task will wake up, when the event is set later. Multiple tasks can wait for the event to be set. This method returns ``TRUE``, if ``nMicroSeconds`` microseconds have elapsed, before the event has been set. To determine, what caused the method to return, use ``GetState()`` to see, if the event has been set. It is possible to have timed out and the event is set anyway.
