# spbctf\_4\_x86\_64

<mark style="color:red;">`main():`</mark>

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v5; // [rsp+13h] [rbp-27Dh]
  int i; // [rsp+14h] [rbp-27Ch]
  int j; // [rsp+14h] [rbp-27Ch]
  int k; // [rsp+14h] [rbp-27Ch]
  int v9; // [rsp+18h] [rbp-278h]
  signed int v10; // [rsp+1Ch] [rbp-274h]
  int v11[12]; // [rsp+20h] [rbp-270h] BYREF
  int v12[128]; // [rsp+50h] [rbp-240h] BYREF
  char s[24]; // [rsp+250h] [rbp-40h] BYREF
  int v14; // [rsp+268h] [rbp-28h]
  char v15; // [rsp+26Ch] [rbp-24h]
  unsigned __int64 v16; // [rsp+278h] [rbp-18h]

  v16 = __readfsqword(0x28u);
  v11[0] = 8;
  v11[1] = 7;
  v11[2] = 5;
  v11[3] = 4;
  v11[4] = 1;
  v11[5] = 3;
  v11[6] = 2;
  v11[7] = 6;
  v11[8] = 9;
  v11[9] = 10;
  qmemcpy(s, "cPK}[aYr^@ZZR`C]TBP_\\Y_U", sizeof(s));
  v14 = 1163351423;
  v15 = 0;
  if ( argc > 1 )
  {
    v10 = strlen(argv[1]);
    for ( i = 0; i < v10; ++i )
    {
      v5 = argv[1][i];
      if ( v5 <= 47 || v5 > 57 )
      {
        puts("Only 0-9");
        return 2;
      }
      v12[i] = argv[1][i] - 48;
    }

    bubble_sort_sequence_executor_aka_transposition_performer(v11, v12, (unsigned int)v10);
    v9 = 1;
    
    for ( j = 0; j <= 9; ++j )
      if ( v11[j] != j + 1 ) 
        v9 = 0;
    
    if ( v9 )
    {
      for ( k = 0; k < strlen(s); ++k )
        s[k] ^= argv[1][k % v10];
      printf("Your flag: %s\n", s);
    }
    else
    {
      puts("Hmmm, Not exactly what I was asking for.");
    }
    return 0;
  }
  else
  {
    puts("Go away, lazy student!");
    return 1;
  }
}
```



<mark style="color:red;">`bubble_sort():`</mark>

```c
__int64 __fastcall bubble_sort_sequence_executor_aka_transposition_performer(__int64 a1, __int64 a2, int a3)
{
  __int64 result; // rax
  unsigned int i; // [rsp+1Ch] [rbp-8h]
  int v5; // [rsp+20h] [rbp-4h]

  for ( i = 0; ; ++i )
  {
    result = i;
    if ( (int)i >= a3 )
      break;
    v5 = *(_DWORD *)(4LL * *(int *)(4LL * (int)i + a2) + a1);
    *(_DWORD *)(a1 + 4LL * *(int *)(4LL * (int)i + a2)) = *(_DWORD *)(4 * (*(int *)(4LL * (int)i + a2) + 1LL) + a1);
    *(_DWORD *)(a1 + 4 * (*(int *)(4LL * (int)i + a2) + 1LL)) = v5;
  }
  return result;
}
```
