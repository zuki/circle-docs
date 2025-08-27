.. _analyzing-exceptions:

例外の解析
~~~~~~~~~~

この付録ではシステムアボート例外の解析方法について説明します。この目的のために、セクション :ref:`a-more-complex-program` のプログラム出力を使用します。それは次のようなものです。

.. code-block:: none

	logger: Circle 44.3 started on Raspberry Pi Zero W
	00:00:01.00 timer: SpeedFactor is 1.00
	00:00:01.00 kernel: An exception will occur after 15 seconds from now
	00:00:02.00 kernel: Time is 2
	00:00:03.00 kernel: Time is 3
	00:00:04.00 kernel: Time is 4
	...
	00:00:14.00 kernel: Time is 14
	00:00:15.00 kernel: Time is 15
	00:00:16.00 except: stack[7] is 0xEFBC
	00:00:16.00 except: stack[8] is 0xF04C
	00:00:16.00 except: stack[11] is 0x11304
	00:00:16.00 except: stack[13] is 0x113C4
	00:00:16.00 except: stack[23] is 0x11408
	00:00:16.00 except: stack[25] is 0x108E4
	00:00:16.00 except: stack[31] is 0xE834
	00:00:16.00 except: Prefetch abort (PC 0x500000, FSR 0xD, FAR 0x500000,
		SP 0x237F80, LR 0xEE7C, PSR 0x20000192)

例外の原因となった命令を検出したい場合は、*kernel\*.lst* ファイルを開き、*PC* (Program Counter) のアドレスを検索します。このアドレスはカーネルイメージの外部にある無効なアドレスなので見つけることはできませんが、*LR* (Link Register) には ``TimerHandler()`` が呼び出されたアドレス (0xEE7C) が指定されています。アドレスは *kernel\*.lst* の行の先頭にあり、末尾にコロンが付いています。

.. code-block:: none

        0000edd0 <CTimer::PollKernelTimers()>:
            edd0:	e92d41f0 	push	{r4, r5, r6, r7, r8, lr}
        ...
            ee64:	e3530000 	cmp	r3, #0
            ee68:	0a000011 	beq	eeb4 <CTimer::PollKernelTimers()+0xe4>
            ee6c:	e1a00004 	mov	r0, r4
            ee70:	e5942010 	ldr	r2, [r4, #16]
            ee74:	e594100c 	ldr	r1, [r4, #12]
            ee78:	e12fff33 	blx	r3
        ==> ee7c:	e1a00004 	mov	r0, r4
            ee80:	e3a01014 	mov	r1, #20
        ...

したがって、 ``TimerHandler()`` は指定されたアドレスの前の命令である "blx r3" によって呼び出されたことがわかります。

*stack[]* からリストされたアドレスからバックトレースが可能ですが、表示されたすべてのアドレスが有効でなくても構いません。 *kernel\*.lst* ファイルで最初のアドレスを検索すればよいだけです。そうすれば、どの関数がどの関数から呼び出されたかという情報が得られます。
