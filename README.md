# DLL-Injection
C++ program that injects a dll into a process using it's procid

# Main Function

```c++
bool Inject(DWORD procid, const std::string& dll) {
    HANDLE handle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, procid);

    if (handle == NULL) {
        std::cout << "[!] Failed To Open Proccess";
        return false;
    }

    std::cout << "[+] Opened Proccess";

    LPVOID allocMem = VirtualAllocEx(handle, NULL, dll.size() + 1, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (allocMem == NULL) {
        std::cerr << "[!] Failed to allocate memory";
        CloseHandle(handle);
        return false;
    }

    std::cout << "\n[+] Allocated Mem\n";

    if (!WriteProcessMemory(handle, allocMem, dll.c_str(), dll.size() + 1, NULL)) {
        std::cerr << "[!] Failed to write memory";
        VirtualFreeEx(handle, allocMem, 0, MEM_RELEASE);
        CloseHandle(handle);
        return false;
    }

    FARPROC loadlib = GetProcAddress(GetModuleHandle("kernel32.dll"), "LoadLibraryA");
    if (loadlib == NULL) {
        std::cerr << "[!] Failed to get address of load lib A";
        VirtualFreeEx(handle, allocMem, 0, MEM_RELEASE);
        CloseHandle(handle);
        return false;
    }

    HANDLE ht = CreateRemoteThread(handle, NULL, 0, (LPTHREAD_START_ROUTINE)loadlib, allocMem, 0, NULL);
    if (ht == NULL) {
        std::cerr << "[!] Failed to create remote thread";
        VirtualFreeEx(handle, allocMem, 0, MEM_RELEASE);
        CloseHandle(handle);
        return false;
    }

    WaitForSingleObject(ht, INFINITE);
    VirtualFreeEx(handle, allocMem, 0, MEM_RELEASE);
    CloseHandle(ht);
    CloseHandle(handle);

    return true;
}
```
