# Allocating-individual-pages
```C++
typedef struct _MMPTE
{
	union
	{
		/* 0x0000 */ unsigned __int64 Long;
		/* 0x0000 */ volatile unsigned __int64 VolatileLong;
		/* 0x0000 */ struct _MMPTE_HARDWARE Hard;
		/* 0x0000 */ struct _MMPTE_PROTOTYPE Proto;
		/* 0x0000 */ struct _MMPTE_SOFTWARE Soft;
		/* 0x0000 */ struct _MMPTE_TIMESTAMP TimeStamp;
		/* 0x0000 */ struct _MMPTE_TRANSITION Trans;
		/* 0x0000 */ struct _MMPTE_SUBSECTION Subsect;
		/* 0x0000 */ struct _MMPTE_LIST List;
	} /* size: 0x0008 */ u;
} MMPTE, * PMMPTE; /* size: 0x0008 */
 
typedef PVOID(__fastcall *fMmAllocateIndependentPagesEx)(IN SIZE_T NumberOfBytes, IN int Node, IN QWORD* a3, IN unsigned int a4); // prototype
 
fMmAllocateIndependentPagesEx g_pMmAllocateIndependentPagesEx = NULL;
PVOID KernelBase = grab_module_base("ntoskrnl.exe");
 
 if (!KernelBase)
        return STATUS_SUCCESS;
 
 // 21h2 pattern 
 g_pMmAllocateIndependentPagesEx = (fMmAllocateIndependentPagesEx)img_pattern_search((PCHAR)KernelBase, "\x48\x8B\xC4\x48\x89\x58\x10\x44\x89\x48\x20\x55\x56\x57\x41\x54\x41\x55\x41\x56\x41\x57\x48\x81\xEC\x00\x00\x00\x00", "xxxxxxxxxxxxxxxxxxxxxxxxx????");
 
 if (g_pMmAllocateIndependentPagesEx)
 {
      PBYTE base = g_pMmAllocateIndependentPagesEx(ntHeaders->OptionalHeader.SizeOfImage, -1, 0i64, 0); // allocating individual pages for driver
      if (!base) {
          DebuggerPrint("You have not allocated lots of memory");
          return STATUS_NO_MEMORY;
      }
 
      UINT number_of_pages = BYTES_TO_PAGES(ntHeaders->OptionalHeader.SizeOfImage); // getting number from your driver image
      DebuggerPrint("You have allocated lots of memory");
 
      PMMPTE PTE = get_pte_addy(base); // your function to grab Page table Entry for allocated memory base
      PMMPTE EndOfPTE = PTE + number_of_pages;
 
      for (PMMPTE pPTE = PTE; pPTE < EndOfPTE; ++pPTE)
      {
           set_pte_bits(pPTE); // basically changing bits(most importantly setting each page as executable so it can run without a problem)
      }
  }
```
