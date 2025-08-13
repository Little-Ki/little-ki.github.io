# 简介
[`ObRegisterCallbacks`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-obregistercallbacks)是一个内核函数，用于创建回调来处理对线程、进程以及桌面句柄的操作。

此函数常被用在反作弊以及反病毒软件。

特别的，反作弊软件的内核驱动使用此方式剥离用户模式下的`OpenProcess`访问权限来阻止内存写入及读取。

## 函数签名
```c
NTSTATUS ObRegisterCallbacks(
  [in]  POB_CALLBACK_REGISTRATION CallbackRegistration,
  [out] PVOID                     *RegistrationHandle
);
```

## 结构
```c
typedef struct _OB_CALLBACK_REGISTRATION {
  USHORT                    Version;
  USHORT                    OperationRegistrationCount;
  UNICODE_STRING            Altitude;
  PVOID                     RegistrationContext;
  OB_OPERATION_REGISTRATION *OperationRegistration;
} OB_CALLBACK_REGISTRATION, *POB_CALLBACK_REGISTRATION;

typedef struct _OB_OPERATION_REGISTRATION {
  POBJECT_TYPE                *ObjectType;
  OB_OPERATION                Operations;
  POB_PRE_OPERATION_CALLBACK  PreOperation;
  POB_POST_OPERATION_CALLBACK PostOperation;
} OB_OPERATION_REGISTRATION, *POB_OPERATION_REGISTRATION;
```

## 其他函数
```c
// 返回进程的ID
HANDLE PsGetProcessId(_In_ PEPROCESS Process);

// 返回具有指定线程的进程
PEPROCESS IoThreadToProcess(_In_ PETHREAD Thread);
```

# 通过限制
使用此函数的内核驱动程序必须具有合法签名，但有一些方式可以绕过。

## 方法一
通过分析，Windows使用`MmVerifyCallbackFunction`检验回调。
此方法只是检查
```c
if (DriverObject->Driversection->Flags ＆ 0x20)
```

``` 
nt!MmVerifyCallbackFunction+0x75: 
fffff800`01a66865 f6406820   test    byte ptr [rax+68h],20h 
fffff800`01a66869 0f45fd     cmovne  edi,ebp
```

`DriverSection`是指向`LDR_DATA_TABLE_ENTRY`的指针
```c
typedef struct _LDR_DATA_TABLE_ENTRY {
    LIST_ENTRY InLoadOrderLinks;
    LIST_ENTRY InMemoryOrderLinks;
    LIST_ENTRY InInitializationOrderLinks;
    PVOID DllBase;
    PVOID EntryPoint;
    ULONG SizeOfImage;
    UNICODE_STRING FullDllName;
    UNICODE_STRING BaseDllName;
    ULONG Flags;
    USHORT LoadCount;
    USHORT TlsIndex;
    union {
        LIST_ENTRY HashLinks;
        struct {
            PVOID SectionPointer;
            ULONG CheckSum;
        };
    };
    union {
        ULONG TimeDateStamp;
        PVOID LoadedImports;
    };
    PVOID EntryPointActivationContext;
    PVOID PatchInformation;
    LIST_ENTRY ForwarderLinks;
    LIST_ENTRY ServiceTagLinks;
    LIST_ENTRY StaticLinks;
} LDR_DATA_TABLE_ENTRY, * PLDR_DATA_TABLE_ENTRY;
```

<details>
<summary>使用如下代码：</summary>

```c
BOOLEAN BypassCheckSign(PDRIVER_OBJECT pDriverObject) {
    #ifdef _WIN64
    typedef struct _KLDR_DATA_TABLE_ENTRY {
        LIST_ENTRY listEntry;
        ULONG64 __Undefined1;
        ULONG64 __Undefined2;
        ULONG64 __Undefined3;
        ULONG64 NonPagedDebugInfo;
        ULONG64 DllBase;
        ULONG64 EntryPoint;
        ULONG SizeOfImage;
        UNICODE_STRING path;
        UNICODE_STRING name;
        ULONG Flags;
        USHORT LoadCount;
        USHORT __Undefined5;
        ULONG64 __Undefined6;
        ULONG CheckSum;
        ULONG __padding1;
        ULONG TimeDateStamp;
        ULONG __padding2;
    }    KLDR_DATA_TABLE_ENTRY, * PKLDR_DATA_TABLE_ENTRY;
    #else
    typedef struct _KLDR_DATA_TABLE_ENTRY {
        LIST_ENTRY listEntry;
        ULONG unknown1;
        ULONG unknown2;
        ULONG unknown3;
        ULONG unknown4;
        ULONG unknown5;
        ULONG unknown6;
        ULONG unknown7;
        UNICODE_STRING path;
        UNICODE_STRING name;
        ULONG Flags;
    }    KLDR_DATA_TABLE_ENTRY, * PKLDR_DATA_TABLE_ENTRY;
    #endif
    PKLDR_DATA_TABLE_ENTRY pLdrData = (PKLDR_DATA_TABLE_ENTRY) pDriverObject->DriverSection;
    pLdrData->Flags = pLdrData->Flags | 0x20;
    return (TRUE);
}
```

</details>

## 方法二
为了使用此方法，PE头中的`IMAGE_OPTIONAL_HEADER.DllCharacterisitics`应该具有`IMAGE_DLLCHARACTERISITICS_FORCE_INTEGRITY `属性
在 Visual Studio 中：

1. 右键单击项目并选择属性
2. 在配置属性中选择链接器，然后单击命令行。
3. 在其他选项中： /INTEGRITYCHECK 设置，/INTEGRITYCHECK:NO 不设置
4. 对驱动进行签名

# 使用示例

<details>
<summary>查看</summary>

```c
/* 注册回调 */

NTSTATUS SetProcessCallbacks()
{
	NTSTATUS			status		= STATUS_SUCCESS;
	OB_CALLBACK_REGISTRATION	obCallbackReg	= { 0 };
	OB_OPERATION_REGISTRATION	obOperationReg	= { 0 };
	RtlZeroMemory( &obCallbackReg, sizeof(OB_CALLBACK_REGISTRATION) );
	RtlZeroMemory( &obOperationReg, sizeof(OB_OPERATION_REGISTRATION) );
	/* set OB_CALLBACK_REGISTRATION */
	obCallbackReg.Version				= ObGetFilterVersion();
	obCallbackReg.OperationRegistrationCount	= 1;
	obCallbackReg.RegistrationContext		= NULL;
	RtlInitUnicodeString( &obCallbackReg.Altitude, L"321000" );
	obCallbackReg.OperationRegistration = &obOperationReg;
	/*
	 * set OB_OPERATION_REGISTRATION
	 * difference between Thread and Process
	 */
	obOperationReg.ObjectType	= PsProcessType;
	obOperationReg.Operations	= OB_OPERATION_HANDLE_CREATE | OB_OPERATION_HANDLE_DUPLICATE;
	/* difference between Thread and Process */
	obOperationReg.PreOperation = (POB_PRE_OPERATION_CALLBACK) (&ProcessPreCall);
	/* 注册回调函数 */
	status = ObRegisterCallbacks( &obCallbackReg, &g_obProcessHandle );
	if ( !NT_SUCCESS( status ) )
	{
		DbgPrint( "ObRegisterCallbacks Error[0x%X]\n", status );
		return(status);
	}
	return(status);
}

NTSTATUS SetThreadCallbacks()
{
	NTSTATUS			status		= STATUS_SUCCESS;
	OB_CALLBACK_REGISTRATION	obCallbackReg	= { 0 };
	OB_OPERATION_REGISTRATION	obOperationReg	= { 0 };
	RtlZeroMemory( &obCallbackReg, sizeof(OB_CALLBACK_REGISTRATION) );
	RtlZeroMemory( &obOperationReg, sizeof(OB_OPERATION_REGISTRATION) );
	/* set OB_CALLBACK_REGISTRATION */
	obCallbackReg.Version				= ObGetFilterVersion();
	obCallbackReg.OperationRegistrationCount	= 1;
	obCallbackReg.RegistrationContext		= NULL;
	RtlInitUnicodeString( &obCallbackReg.Altitude, L"321001" );
	obCallbackReg.OperationRegistration = &obOperationReg;
	/*
	 * set OB_OPERATION_REGISTRATION
	 * difference between Thread and Process
	 */
	obOperationReg.ObjectType	= PsThreadType;
	obOperationReg.Operations	= OB_OPERATION_HANDLE_CREATE | OB_OPERATION_HANDLE_DUPLICATE;
	/* difference between Thread and Process */
	obOperationReg.PreOperation = (POB_PRE_OPERATION_CALLBACK) (&ThreadPreCall);
	/* regist call back */
	status = ObRegisterCallbacks( &obCallbackReg, &g_obThreadHandle );
	if ( !NT_SUCCESS( status ) )
	{
		DbgPrint( "ObRegisterCallbacks Error[0x%X]\n", status );
		return(status);
	}
	return(status);
}

OB_PREOP_CALLBACK_STATUS ThreadPreCall( PVOID RegistrationContext, POB_PRE_OPERATION_INFORMATION pObPreOperationInfo )
{
	PEPROCESS pEProcess = NULL;
	/* check object type */
	if ( *PsThreadType != pObPreOperationInfo->ObjectType )
	{
		return(OB_PREOP_SUCCESS);
	}
	/* get thread owner PEPROCESS */
	pEProcess = IoThreadToProcess( (PETHREAD) pObPreOperationInfo->Object );
	/* check if PID should be protected */
	if ( IsProtectProcess( pEProcess ) )
	{
		/* operation type: create handle */
		if ( OB_OPERATION_HANDLE_CREATE == pObPreOperationInfo->Operation )
		{
			if ( 1 == (1 & pObPreOperationInfo->Parameters->CreateHandleInformation.OriginalDesiredAccess) )
			{
				pObPreOperationInfo->Parameters->CreateHandleInformation.DesiredAccess = 0;
			}
		}
		/* operation type: duplicate handle */
		else if ( OB_OPERATION_HANDLE_DUPLICATE == pObPreOperationInfo->Operation )
		{
			if ( 1 == (1 & pObPreOperationInfo->Parameters->DuplicateHandleInformation.OriginalDesiredAccess) )
			{
				pObPreOperationInfo->Parameters->DuplicateHandleInformation.DesiredAccess = 0;
			}
		}
	}
	return(OB_PREOP_SUCCESS);
}

OB_PREOP_CALLBACK_STATUS ProcessPreCall( PVOID RegistrationContext, POB_PRE_OPERATION_INFORMATION pObPreOperationInfo )
{
	PEPROCESS pEProcess = NULL;
	/* check object type */
	if ( *PsProcessType != pObPreOperationInfo->ObjectType )
	{
		return(OB_PREOP_SUCCESS);
	}
	/* get process object */
	pEProcess = (PEPROCESS) pObPreOperationInfo->Object;
	/* check if PID should be protected */
	if ( IsProtectProcess( pEProcess ) )
	{
		/* operation type: create handle */
		if ( OB_OPERATION_HANDLE_CREATE == pObPreOperationInfo->Operation )
		{
			if ( 1 == (1 & pObPreOperationInfo->Parameters->CreateHandleInformation.OriginalDesiredAccess) )
			{
				pObPreOperationInfo->Parameters->CreateHandleInformation.DesiredAccess = 0;
			}
		}
		/* operation type: duplicate handle */
		else if ( OB_OPERATION_HANDLE_DUPLICATE == pObPreOperationInfo->Operation )
		{
			if ( 1 == (1 & pObPreOperationInfo->Parameters->DuplicateHandleInformation.OriginalDesiredAccess) )
			{
				pObPreOperationInfo->Parameters->DuplicateHandleInformation.DesiredAccess = 0;
			}
		}
	}
	return(OB_PREOP_SUCCESS);
}
```

</details>

# 绕过

## 多种方法

1. 将`_OBJECT_TYPE_INITIALIZER.RetainAccess`的成员`PsProcessType`和`PsThreadType`设置为0x1fffff
2. 将`ETHREAD.PreviousMode`设置为kernelMode
3. 枚举`EPROCESS->HandleTable`并将`GransAccess`设置为0x1fffff
4. 将代理DLL注入csrss.s.exe / lsass.exe或其他系统检查，然后使用现有句柄
5. 枚举回调例程列表然后编辑它

关于第5条：操作系统维护了一个列表，此列表包含所有已注册的回调信息，如函数指针，修改它可以绕过。
列表的表头在`OBJECT_TYPE`的结构体中：
```c
typedef struct OBJECT_TYPE {
    ...
    PLIST_ENTRY64 header;
    ...
}
```

在WinDBG中有：
```
kd> dt nt!_OBJECT_TYPE
+0x000 TypeList         : _LIST_ENTRY [ 0xffffcb82`dee6cf20 - 0xffffcb82`dee6cf20 ]
+0x010 Name             : _UNICODE_STRING "Process"
+0x020 DefaultObject    : (null) 
+0x028 Index            : 0x7 ''
+0x02c TotalNumberOfObjects : 0x26
+0x030 TotalNumberOfHandles : 0xe8
+0x034 HighWaterNumberOfObjects : 0x26
+0x038 HighWaterNumberOfHandles : 0xea
+0x040 TypeInfo         : _OBJECT_TYPE_INITIALIZER
+0x0b8 TypeLock         : _EX_PUSH_LOCK
+0x0c0 Key              : 0x636f7250
+0x0c8 CallbackList     : _LIST_ENTRY [ 0xffffa002`d31bacd0 - 0xffffa002`d35d2450 ]
```

`CallbackList`有以下结构：
```c
 typedef struct _CALLBACK_ENTRY_ITEM {
     LIST_ENTRY EntryItemList;
     OB_OPERATION Operations;
     CALLBACK_ENTRY * CallbackEntry;
     POBJECT_TYPE ObjectType;
     POB_PRE_OPERATION_CALLBACK PreOperation;
     POB_POST_OPERATION_CALLBACK PostOperation;
     __int64 unk;
 } CALLBACK_ENTRY_ITEM, * PCALLBACK_ENTRY_ITEM;
``` 

符号`PsProcessType`和`PsthreadType`已导出，通过偏移可以获取列表头（注：指针指向表头的内部成员而非头部）
该成员的偏移尚未固定，在不同版本的系统中可能有所不同。