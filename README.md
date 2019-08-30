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
`

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
`

这里的nProcNum=3

![image](https://github.com/laomms/SLInstallProofOfPurchase/blob/master/1.png)

条用的dll是C:\Windows\System32\combase.dll
这里总共调用了5个函数，其中第三个函数的相对地址是:0x00007fffb51a1270，可以尝试直接跳转该地址下断点试试。
```c#
[
uuid(18f70770-8e64-11cf-9af1-0020af6e72f4),
version(0.0),
]
interface DefaultIfName
{

	typedef struct Struct_30_t
	{
		short 	StructMember0;
		short 	StructMember1;
	[size_is(StructMember0)]short StructMember2[];
		}Struct_30_t;

error_status_t Proc0(
	[in]short arg_1, 
	[in][unique][string] wchar_t* arg_2, 
	[out]long *arg_3, 
	[out][ref]struct Struct_30_t** arg_4, 
	[out][ref]struct Struct_30_t** arg_5);

error_status_t Proc1(
	[in]short arg_1, 
	[in][size_is(arg_1)]short arg_2[], 
	[out][ref]struct Struct_30_t** arg_3);

error_status_t Proc2(                   //函数结构
	[in]struct Struct_30_t* arg_2, 
	[in][out]hyper *arg_3, 
	[out][ref]struct Struct_30_t** arg_4, 
	[out][ref]struct Struct_30_t** arg_5);

error_status_t Proc3(
	[in]unsigned __int3264 arg_1);

error_status_t Proc4(
	[in]long arg_1);
} 
`

IDA载入combase.dll再做分析
