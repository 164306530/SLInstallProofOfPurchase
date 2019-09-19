# SLInstallProofOfPurchase
```c
HRESULT SLInstallProofOfPurchase(
  HSLC   hSLC,
  PCWSTR pwszPKeyAlgorithm,
  PCWSTR pwszPKeyString,
  UINT   cbPKeySpecificData,
  PBYTE  pbPKeySpecificData,
  SLID   *pPkeyId
);
```

想知道PKeyId是怎么产生的，目的是想不通过安装密钥得到PKeyId,所以调试了下SLInstallProofOfPurchase.
```c
__int64 __fastcall SLInstallProofOfPurchase(__int64 hSLC, __int64 Algorithm, __int64 Key, int cbPKeySpecificData, __int64 pbPKeySpecificData, __int64 PkeyId)
{
  __int64 slc; // rbp
  __int64 szkey; // rsi
  __int64 szAlgorithm; // rbx
  __int64 v9; // rcx
  unsigned int v10; // ebx
  signed int v11; // eax
  __int64 v12; // rdx
  unsigned int v13; // eax
  char v15; // [rsp+30h] [rbp-28h]
  int SpecificData; // [rsp+78h] [rbp+20h]

  SpecificData = cbPKeySpecificData;
  slc = hSLC;
  szkey = Key;
  szAlgorithm = Algorithm;
  sub_7FFFB15D59C4(&v15);
  if ( !slc || !szAlgorithm || !szkey || !PkeyId )
    goto LABEL_21;
  v11 = sub_7FFFB15E0FDC(&v15, 1i64, szAlgorithm);
  v10 = v11;
  if ( v11 < 0 )
    goto LABEL_8;
  v11 = sub_7FFFB15E0FDC(&v15, 2i64, szkey);
  v10 = v11;
  if ( v11 < 0 )
    goto LABEL_8;
  v11 = sub_7FFFB15D51E0(&v15, 3i64, &SpecificData);
  v10 = v11;
  if ( v11 < 0 )
    goto LABEL_8;
  if ( !SpecificData )
    goto LABEL_14;
  if ( !pbPKeySpecificData )
  {
LABEL_21:
    v9 = 2147942487i64;
    v10 = -2147024809;
LABEL_3:
    sub_7FFFB15D8640(v9);
    goto LABEL_18;
  }
  v11 = sub_7FFFB15E0FAC(&v15, 4i64);
  v10 = v11;
  if ( v11 < 0 )
  {
LABEL_8:
    v9 = (unsigned int)v11;
    goto LABEL_3;
  }
LABEL_14:
  v10 = sub_7FFFB15D4804((__int64 *)&v15, slc, 2u, 1, 1);//跟入
  ...
  
   v10.Pointer = LPC_Sub(v6, v5, v13, (__int64)hMem, (__int64)&v14, (__int64)&v16).Pointer;
  v8 = (unsigned int)v10.Pointer;
  if ( SLODWORD(v10.Pointer) < 0 )
  {
    sub_7FFFB15D8640(LODWORD(v10.Pointer));
    goto LABEL_11;
  }
  ...
  
  CLIENT_CALL_RETURN __fastcall LPC_Sub(__int64 a1, int a2, int a3, __int64 a4, __int64 a5, __int64 a6)
{
  int v7; // [rsp+20h] [rbp-38h]
  int v8; // [rsp+28h] [rbp-30h]

  v8 = a3;
  v7 = a2;
  return NdrClientCall3(&pProxyInfo, 3u, 0i64, a1, v7, v8, a4, a5, a6);// RPC客户端调用,通过RpcViewer查看调用的是哪个DLL及函数
  //如果成功,服务方会送回20个字节的数据，其中就包括PKeyID
 a5:
 0x00000001033FDA44  20 00 00 00 00 88 bc 5f 68 01 00 00 d0 66 6f 65   ....??_h...?foe
 a6
 0x00000001033FDA50  d0 66 6f 65 68 01 00 00 00 00 00 00 00 00 00 80  ?foeh..........€   //PkeyID存储地址
 
 0x00000168656f66d0
 0x00000168656F66D0  e2 d9 dd 01 04 00 00 00 00 00 00 00 10 00 00 00  ???.............
 0x00000168656F66E0  a9 2d 48 7c 5e 25 3c a8 ab cd 16 96 1a 56 83 21  ?-H|^%<???.?.V?!  //pkeyID
  
}
`
```c#
    struct MIDL_STUBLESS_PROXY_INFO   //(sizeof=0x30, align=0x8, copyof_25)
    {
        IntPtr pStubDesc;  //RPC客户端接口
        IntPtr ProcFormatString;
        IntPtr FormatStringOffset;
        IntPtr pTransferSyntax;
        IntPtr nCount;
        IntPtr pSyntaxInfo;
    }
    
     [DllImport("RpcRT4.dll", EntryPoint = "NdrClientCall3", ExactSpelling = false, CharSet = CharSet.Unicode)]
        private extern static int NdrClientCall3(MIDL_STUBLESS_PROXY_INFO pProxyInfo, ulong nProcNum, IntPtr pReturnValue, IntPtr a1, int a2, int a3, IntPtr a4, IntPtr a5, IntPtr a6);


pStubDesc结构
0x000007FEF0CA1F80  90 1d ca f0 fe 07 00 00 44 0c cb f0 fe 07 00 00  ?.???...D.???...
0x000007FEF0CA1F90  54 0c cb f0 fe 07 00 00 08 12 cc f0 fe 07 00 00  T.???.....???...
0x000007FEF0CA1FA0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

看下000007FEF0CA1D90内存地址:
0x000007FEF0CA1D90  60 00 00 00 56 cc 35 94 9c 1d 24 49 ac 7d b6 0a  `...V?5??.$I?}?.
0x000007FEF0CA1DA0  2c 35 20 e1 01 00 00 00 04 5d 88 8a eb 1c c9 11  ,5 ?.....]???.?.
0x000007FEF0CA1DB0  9f e8 08 00 2b 10 48 60 02 00 00 00 00 00 00 00  ??..+.H`........
这里可以看到InterfaceId的GUID为9435cc56-1d9c-4924-ac7d-b60a2c3520e1

```
rpcview中查找这个GUID值:

![image](https://github.com/laomms/SLInstallProofOfPurchase/blob/master/1.png)

可以看到rpc服务端为 C:\Windows\System32\sppsvc.exe
返回函数的偏移地址为4E700, 函数结构为
```c#
[
uuid(9435cc56-1d9c-4924-ac7d-b60a2c3520e1),
version(1.0),
]
interface DefaultIfName
{

long Proc0(
	[out][context_handle] void** arg_1);

long Proc1(
	[in][out][context_handle] void** arg_0);

long Proc2(
	[in][context_handle] void* arg_0, 
	[in]/* enum_16 */ short arg_1, 
	[in][range(0,5242880)] long arg_2, 
	[in][size_is(arg_2)]char arg_3[], 
	[out]long *arg_4, 
	[out][ref][size_is(, *arg_4)]char **arg_5);

long Proc3(
	[in][context_handle] void* arg_0, 
	[in]/* enum_16 */ short arg_1, 
	[in][range(0,5242880)] long arg_2, 
	[in][size_is(arg_2)]char arg_3[], 
	[out]long *arg_4, 
	[out][ref][size_is(, *arg_4)]char **arg_5);
} 

```
IDA载入sppsvc.exe,找到偏移地址:
![image](https://github.com/laomms/SLInstallProofOfPurchase/blob/master/2.png)
```c
__int64 __fastcall CSppServiceOperationT<CEmptyType>::Phase2InitSecureCallback(RPC_IF_HANDLE InterfaceUuid, void *Context)
{
  RPC_STATUS v2; // eax
  int v3; // eax
  unsigned int v4; // ebx
  RPC_STATUS v5; // eax
  unsigned int v6; // ecx
  __int64 *v8; // [rsp+20h] [rbp-58h]
  __int64 v9; // [rsp+28h] [rbp-50h]
  int RpcCallAttributes; // [rsp+30h] [rbp-48h]
  char Dst; // [rsp+34h] [rbp-44h]
  int v12; // [rsp+58h] [rbp-20h]
  unsigned int Pid; // [rsp+90h] [rbp+18h]
  __int64 v14; // [rsp+98h] [rbp+20h]

  memset(&Dst, 0, 0x34ui64);
  RpcCallAttributes = 1;
  v2 = RpcServerInqCallAttributesW(0i64, &RpcCallAttributes);
  if ( v2 )
  {
    v3 = I_RpcMapWin32Status(v2);
    v4 = (unsigned __int16)v3 | 0x80070000;
    if ( v3 <= 0 )
      v4 = v3;
    if ( (v4 & 0x80000000) != 0 )
      goto LABEL_16;
  }
  if ( v12 != 6 )
  {
    v4 = -2147024891;
LABEL_16:
    if ( WPP_GLOBAL_Control != (WARBIRD *)&WPP_GLOBAL_Control && *((_BYTE *)WPP_GLOBAL_Control + 60) & 2 )
      WPP_SF_d(*((_QWORD *)WPP_GLOBAL_Control + 6), 25i64, qword_100021560, v4);
    return v4;
  }
  if ( CSppServiceOperationT<CEmptyType>::g_fPhase2InitAttempted )
  {
    v4 = CSppServiceOperationT<CEmptyType>::g_hrInitStatus;
  }
  else
  {
    v5 = I_RpcBindingInqLocalClientPID(0i64, &Pid);
    v6 = Pid;
    if ( v5 )
      v6 = 0;
    Pid = v6;
    LODWORD(v14) = v6;
    if ( SLEtwHelperT<CEmptyType>::g_SLTraceRegHandle
      && (unsigned __int8)SLEtwHelperT<CEmptyType>::pfnEventEnabled(
                            SLEtwHelperT<CEmptyType>::g_SLTraceRegHandle,
                            &SLSVC_External_ExternalCallDuringInit) )
    {
      v8 = &v14;
      v9 = 4i64;
      ((void (__fastcall *)(_QWORD, _QWORD *, __int64, __int64 **))SLEtwHelperT<CEmptyType>::pfnEventWrite)(
        SLEtwHelperT<CEmptyType>::g_SLTraceRegHandle,
        &SLSVC_External_ExternalCallDuringInit,
        1i64,
        &v8);
    }
    v4 = CSppServiceOperationT<CEmptyType>::Phase2InitWorker(0i64);
  }
  if ( (v4 & 0x80000000) != 0 )
    goto LABEL_16;
  return v4;
}
```


32位的调用的是NdrClientCall2:
```c
private extern static int NdrClientCall4(IntPtr pStubDescriptor, IntPtr pFormat, IntPtr Handle, int DataSize, int Data, IntPtr ResponseSize, IntPtr Response,IntPtr a6);

a6:
0x010FE488  c0 c0 8f 0a c0 80 4a 01 00 00 00 00 6c e4 0f 01  ???.?€J.....l?..
0a8fc0c0:
0x0A8FC0C0  e2 d9 dd 01 04 00 00 00 00 00 00 00 10 00 00 00  ???.............
0x0A8FC0D0  a9 2d 48 7c 5e 25 3c a8 ab cd 16 96 1a 56 83 21  ?-H|^%<???.?.V?!   //pkeyid

根据pStubDescriptor结构查询InterfaceId的uuid:
0x6F881008  a8 20 88 6f 30 21 89 6f 60 21 89 6f 28 60 89 6f  ? ?o0!?o`!?o(`?o
0x6F881018  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

0x6f8820a8
0x6F8820A8  44 00 00 00 56 cc 35 94 9c 1d 24 49 ac 7d b6 0a  D...V?5??.$I?}?.
0x6F8820B8  2c 35 20 e1 01 00 00 00 04 5d 88 8a eb 1c c9 11  ,5 ?.....]???.?.
0x6F8820C8  9f e8 08 00 2b 10 48 60 02 00 00 00 00 00 00 00  ??..+.H`........
此次InterfaceId GUID为 9435cc56-1d9c-4924-7dac-e120352c0ab6，可以用于查询rpcview
```

换API monitor监视下客户端的调用过程:
![image](https://github.com/laomms/SLInstallProofOfPurchase/blob/master/12.png)

从"ncalrpc"看出RPC的模式是Local Procedure Call（LPC）模式,即ALPC.

```c
服务端代码：
server.c
--------------
RpcServerUseProtseqEp(
    (unsigned char *)"ncalrpc",
    RPC_C_PROTSEQ_MAX_REQS_DEFAULT,
    (unsigned char *)"AppName",
    NULL);


客户端代码：
--------------
// 用LPC 方式通信
// 第3 个参数NetworkAddr 只能取NULL
RpcStringBindingCompose(
    NULL,
    (unsigned char*)"ncalrpc",
    NULL, (unsigned char*)"AppName",
    NULL,
    &pszStringBinding );
```
再用Process Monitor看一下
![image](https://github.com/laomms/SLInstallProofOfPurchase/blob/master/13.png)

