.. _rename:

RENAME
=======

.. function:: RENAME key newkey

将\ ``key``\ 改名为\ ``newkey``\ 。

当\ ``key``\ 和\ ``newkey``\ 相同或者\ ``key``\ 不存在时，返回一个错误。

当\ ``newkey``\ 已经存在时，\ `RENAME`_\ 命令将覆盖旧值。

**时间复杂度：**
    O(1)

**返回值：**
    改名成功时提示\ ``OK``\ ，失败时候返回一个错误。

:: 

    # 情况1：key存在且newkey不存在

    redis> SET message "hello world"
    OK
    
    redis> RENAME message greeting
    OK

    redis> EXISTS message  # message不复存在
    (integer) 0
    
    redis> EXISTS greeting  # greeting取而代之
    (integer) 1


    # 情况2：当key不存在时，返回错误
    
    redis> RENAME fake_key never_exists
    (error) ERR no such key
    

    # 情况3：newkey已存在时，RENAME会覆盖旧newkey
    
    redis> SET pc "lenovo"
    OK
    
    redis> SET personal_computer "dell"
    OK

    redis> RENAME pc personal_computer
    OK

    redis> GET pc
    (nil)

    redis:1> GET personal_computer  # dell“没有”了
    "lenovo"



