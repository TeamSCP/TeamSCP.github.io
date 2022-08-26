---
title: "메모장 WriteFile() API 후킹"

author:

    name: 신재형

categories:

    - Reversing

tags:

    - [APIHooking]



date: 2022-08-27

last_modified_at: 2022-08-27

---

# 후킹이란?

리버싱에서 후킹은 정보를 가로채고, 실행 흐름을 변경하고, 원래와는 다른 기능을 제공하게 하는 기술입니다.

후킹의 전체 프로세스

- 디스어셈블러/디버거를 이용하여 프고그램의 구조와 동작 원리를 파악

- 버그 수정 또는 기능 개선에 필요한 훅(Hook) 코드를 개발

- 실행 파일과 프로세스 메모리를 자유롭게 조작하여 훅 코드 설치

여러가지 후킹 중 Win32 API를 후킹하는 기술을 API 후킹이라고 합니다.

---

# API란?

Windows OS에서 시스템 자원(메모리, 파일, 네트워크 등)은 OS가 직접 관리하고 여러가지 이유(안정성, 보안, 효율 등)로 인해 사용자 애플리케이션의 직접적인 접근을 막아놓았습니다.

사용자 애플리케이션이 시스템 자원을 사용하기 위해서는 시스템 커널에게 요청해야 하는데 요청 방법이 MS에서 제공한 Win32 API를 이용하는 것입니다.

즉 API 함수 없이는 시스템 자원에 접근할 수 없기 때문에 의미 있는 프로그램을 만들어낼 수 없습니다.

![후킹1](https://user-images.githubusercontent.com/101708067/186912076-251560de-cb3f-4317-bff8-f09c85d1387a.jpg)

모든 프로세스에는 기본적으로 kernel32.dll이 로딩되며, kernel32.dll은 ntdll.dll을 로딩합니다.

- 특정 시스템 프로세스(smss.exe)의 경우 kernel32.dll을 로딩하지 않습니다.

ntdll.dll은 유저 모드 애플리케이션의 코드에서 발생하는 시스템 자원에 대한 접근을 커널 모드에게 요청하는 역할을 수행합니다.

- kernel32.dll → ntdll.dll

---

# API 후킹이란?

Win32 API 호출을 중간에서 가로채어 제어권을 얻어내는 것

- API 호출 전/후에 사용자의 훅 코드를 실행시킬 수 있습니다.

- API에 넘어온 파라미터 혹은 API 함수의 리턴 값을 엿보거나 조작할 수 있습니다.

- API 호출 자체를 취소시키거나 사용자 코드로 실행 흐름을 변경시킬 수 있습니다.

---

# API 후킹의 테크 맵

## ![테크맵](https://user-images.githubusercontent.com/101708067/186912389-669dff37-5c75-4512-b162-f3b33ef926c3.jpg)

## 1. Method Object(what)

API 후킹 방식(Method)에 대한 대분류로 API 후킹 방식은 작업 대상에 따라서 크게 static 방식과 dynamic 방식으로 나눌 수 있습니다.

| Static       | Dynamic              |
| ------------ | -------------------- |
| 파일 대상        | 메모리 대상               |
| 프로그램 실행 전 후킹 | 프로그램 실행 후 후킹         |
| 최초 한 번만 후킹   | 실행될 때마다 후킹           |
| 특수한 상황에서 사용됨 | 일반적인 후킹 방법           |
| Unhook 불가능   | 프로그램 실행 중에 Unhook 가능 |

## 2. Location(where)

### API 후킹을 위한 공략 위치

1. IAT

IAT에 있는 API 주소를 후킹 함수로 변경하는 방법입니다.

장점 : 구현 방법 쉬움

단점 : IAT에 없는데 프로그램에서 사용되는 API들에 대해서 후킹 불가능(예 : DLL을 동적으로 로딩해서 사용하는 경우)

2. Code

프로세스 메모리에 매핑된 시스템 라이브러리(*.dll)에서 API의 실제 주소를 찾아가 코드를 직접 수정해버리는 방법입니다.

이 방법이 가장 널리 사용되는 방법이며 다음과 같은 여러가지 다양한 옵션이 존재합니다.

- 시작 코드를 JMP 명령어로 패치

- 함수 일부 덮어쓰기

- 필요한 부분만 일부 변경
3. EAT

DLL의 EAT에 기록된 API의 시작 주소를 후킹 함수 주소로 변경하는 방법입니다.

하지만 앞서 설명한 Code의 방법이 더 간단하고 강력하므로 EAT 수정 방법은 잘 사용되지 않습니다.

## 3. Technique(How)

후킹 대상 프로세스 메모리에 침투하여 후킹 함수를 설치하는 구체적 기법입니다.

크게 디버그 기법과 인젝션 기법으로 나눌 수 있습니다.

1. 디버그 기법

대상 프로세스를 디버깅하면서 API 후킹을 하는 방법입니다.

디버거는 디버깅을 당하는 프로세스인 디버기에 대한 모든 권한을 가지기 때문에, 디버기의 프로세스 메모리에 후킹 함수를 자유롭게 설치할 수 있습니다.

여기서의 디버거는 일반적인 올리디버거, x64디버거, IDA와 같은 프로그램이 아니라 후킹을 위해 사용자가 직접 제작한 프로그램을 뜻합니다. 즉 프로그램에서 Debug API를 이용하여 대상 프로세스에 Attach하고 후킹 함수를 설치합니다. 그런 후에 실행을 재개하면 API 후킹이 이뤄집니다.

2. 인젝션 기법

인젝션 기법은 해당 프로세스 메모리 영역에 침투하는 기술로, 인젝션 대상에 따라 DLL 인젝션과 Code 인젝션으로 나눌 수 있습니다.

- DLL 인젝션
  
  - DLL 인젝션 기법은 대상 프로세스로 하여금 강제로 사용자가 원하는 DLL 파일을 로딩하게 만드는 기술입니다.
  
  - 인젝션할 DLL에 미리 후킹 코드와 설치 코드를 만들고 DllMain()에서 설치 코드를 호출해주면, 인젝션 되는 순간 API 후킹이 완료됩니다.

- Code 인젝션
  
  - Code 인젝션 기법은 DLL 인젝션보다 좀 더 발전된 기술이며, 주로 악성코드에서 많이 사용됩니다.
  
  - DLL 인젝션 처럼 완전한 형태의 PE Image가 아니라 실행 코드와 데이터만 인젝션된 상태에서 자신이 필요한 API를 직접 구해서 사용해야 하고 코드 내의 메모리 주소에 접근할 때 잘못된 주소를 액세스하지 않도록 주의해야 하기 때문에 Code 인젝션 기법의 구현 방법은 까다로운 편입니다.

## 4. API

테크 맵에 소개된 방법들을 실제로 구현하기 위해 사용되는 API를 보여줍니다.

테크 맵에 있는 API가 아니더라도 OpenProcess(), WriteProcessMemory(), ReadProcessMemory() API 들은 다른 프로세스 메모리에 접근하려고 할 때 항상 사용되는 API입니다.

# 메모장 WriteFile() 후킹

디버거 : 디버깅 프로그램

디버기 : 디버기 당하는 프로그램

디버거 프로세스로 등록되면 OS는 디버기에서 디버그 이벤트가 발생할 때 디버기의 실행을 멈추고 해당 이벤트를 디버거에게 통보합니다. 그리고 디버거는 해당 이벤트에 대해 적절한 처리를 한 후 디버기의 실행을 재개할 수 있습니다.

## 디버그 이벤트 종류

- **EXCEPTION_DEBUG _EVENT**
- CREATE_THREAD_DEBUG_EVENT
- CREATE_PROCESS_DEBUG_EVENT
- EXIT_THREAD_DEBUG_EVENT
- EXIT_PROCESS_DEBUG_EVENT
- LOAD_DLL_DEBUG_EVENT
- UNLOAD_DLL_DEBUG_EVENT
- OUTPUT_DEBUG_STRING_EVENT
- RIP_EVENT

디버그 이벤트 중에서 디버깅에 관련된 이벤트는 첫 번째인 EXCEPTION_DEBUG_EVENT입니다.

이와 관련된 예외 목록은 다음과 같습니다.

- EXCEPTION_ACCESS_VIOLATION
- EXCEPTION_ARRAY BOUNDS_EXCEEDED
- **EXCEPTION_BREAKPOINT**
- EXCEPTION_DATATYPE_MISALIGNMENT
- EXCEPTION_FLT_DENORMAL_OPERAND
- EXCEPTION_FLT_DIVIDE_BY_ZERO
- EXCEPTION_FLT_INEXACT_RESULT
- EXCEPTION_FLT_INVALID_OPERATION
- EXCEPTION_FLT_OVERFLOW
- EXCEPTION_FLT_STACK_CHECK
- EXCEPTION_FLT_UNDERFLOW
- EXCEPTION_ILLEGAL_INSTRUCTION
- EXCEPTION_IN_PAGE_ERROR
- EXCEPTION_INT_DIVIDE_BY_ZERO
- EXCEPTION_INT_OVERFLOW
- EXCEPTION_INV_ALID_DISPOSITION
- EXCEPTION_NONCONTINUABLE_EXCEPTION
- EXCEPTION_PRIV_INSTRUCTION
- EXCEPTION_SINGLE_STEP
- EXCEPTION_STACK_OVERFLOW

이 예외 중 디버거가 반드시 처리해야 하는 예외는 EXCEPTION_BREAKPOINT 예외입니다.

브레이크 포인트는 어셈블리 명령어로 'INT3'이며, IA-32 Instruction으로는 0xCC입니다.

코드 디버깅 중 INT3 명령어를 만나면 실행이 중지되고 디버거에게 EXCEPTION_BREAKPOINT 예외 이벤트가 날아갑니다. 이때 디버거는 다양한 작업을 할 수 있습니다.

## 디버그 기법을 통한 API 후킹

1. 후킹을 원하는 프로세스에 attach하여 디버기를 만듦
2. 훅 : API 시작 주소의 첫 바이트를 0xCC(INT3)로 변경
3. 해당 API가 호출되면 제어는 디버거에게 넘어옴
4. 원하는 작업 수행
5. 언훅 : 0xCC(INT3)를 원래대로 복원 (API의 정상 실행을 위해)
6. 해당 API 실행
7. 훅 : 다시 0xCC로 바꿈 (지속적인 후킹을 위해)
8. 디버기에게 제어를 되돌려줌

## Notepad.exe의 WriteFile() API 후킹

파일이 저장될 때 입력된 파라미터를 조작하여 소문자로 입력된 내용을 전부 대문자로 변경하겠습니다.

먼저 다음과 같이 notepad.exe를 실행시켜 PID가 1180인 것을 확인합니다.

![후킹2](https://user-images.githubusercontent.com/101708067/186912483-afd24a3c-a61b-43aa-8ff4-6362b617eda6.jpg)

이제 다음과 같이 후킹 프로그램(hookdbg.exe)을 실행하고 실행 파라미터로 후킹할 프로세스의 PID를 받습니다.

![후킹3](https://user-images.githubusercontent.com/101708067/186912571-3cf12681-8d26-4130-9efb-6756e62f9b2f.jpg)

이렇게 hookdbg.exe를 실행하면 PID 1180에 해당하는 notepad 프로세스의  WriteFile() API 후킹이 시작됩니다.

메모장에 아무 글자나 입력하고 저장을 해보겠습니다.

![후킹4](https://user-images.githubusercontent.com/101708067/186912578-12884c9c-e960-4880-88f8-dcfc302db338.jpg)

![후킹5](https://user-images.githubusercontent.com/101708067/186912579-35a1b166-e55e-461f-a394-3313e63a83ad.jpg)

저장을 마치면 notepad에는 아무 변화가 일어나지 않습니다. 하지만 notepad를 종료하고 hookdbg 프로그램을 확인해보면 다음과 같이 original string에는 처음 입력한 문자열이 나타나고 converted string에는 WriteFile() API 후킹에 의해 소문자가 대문자로 변경된 문자열이 나타납니다.

![후킹6](https://user-images.githubusercontent.com/101708067/186912583-d3e1a7ad-6ecb-43e0-82d4-bc5153672bd6.jpg)

그리고 저장된 writefilehook.txt 파일을 열면 실제로 모든 소문자가 대문자로 변경되어서 저장된 것을 확인할 수 있습니다.

![후킹7](https://user-images.githubusercontent.com/101708067/186944199-bdee540f-9771-4155-b3b7-19d8090ec5a7.jpg)

## 동작원리

이 동작 원리는 notepad에서 뭔가를 파일에 저장하려면 kernel32!WriteFile() API를 사용할 것이라고 가정합니다.

먼저 WriteFile() API MSDN을 확인해보겠습니다.

![WriteFile() API MSDN](https://user-images.githubusercontent.com/101708067/186944399-2e449365-d5af-4e49-b9ba-01083019b1ad.jpg)

WriteFile() API의 정의를 보면 두 번째 파라미터(lpBuffer)가 '쓰기 버퍼'이고 세 번째 파라미터(nNumberOfBytesToWrite)가 '써야 할 크기'입니다. 그리고 함수의 파라미터는 스택에 역순으로 저장됩니다.

실제로 notepad를 디버깅하면서 확인해보겠습니다.

먼저 올리디버거를 통해 notepad.exe 파일을 열고 kernel32!WriteFile() API를 찾아 BP를 설치한 후 F9를 눌러 실행시킵니다. 그리고 다음과 같이 아무 문자열이나 입력하고 파일을 저장합니다.

![후킹8](https://user-images.githubusercontent.com/101708067/186912586-3cc1e098-80cd-4ac9-af8d-64e824e436fb.jpg)

![후킹9](https://user-images.githubusercontent.com/101708067/186912588-32b7a08d-d2e4-40ba-910e-1307e5fd3118.jpg)

그렇게 되면 다음과 같이 kernel32!WriteFile()에 멈추게 됩니다.

![후킹10](https://user-images.githubusercontent.com/101708067/186912591-212fa050-38ff-42c0-a0f8-bbc7b0284118.jpg)여기서 F7을 눌러 다음과 같이 해당 함수 부분으로 진입합니다.

![후킹11](https://user-images.githubusercontent.com/101708067/186912593-9f938550-7e2b-451c-85e3-f41b34bd1842.jpg)

현재 스택(ESP: 7FACC)에는 리턴 주소(01004C30)가 있고 ESP+8(7FAD8)에는 '쓰기버퍼' 주소(CDE70)가 저장되어 있습니다.

이를 확인하기 위해 덤프창에서 쓰기퍼버 주소(CDE70)으로 이동하면 다음과 같이 저장하려고 하는 문자열(Goodscp)이 보입니다.

![후킹12](https://user-images.githubusercontent.com/101708067/186912596-9b1b6e97-f0eb-40fe-8d6c-fe6588f84316.jpg)

이 WriteFile() API를 후킹해서 쓰기 버퍼를 원하는 문자열로 덮어쓰면 디버그 방법을 사용한 API 후킹 성공입니다.

## hookdbg.exe

```hookdbg.cpp
#include "windows.h"
#include "stdio.h"

LPVOID g_pfWriteFile = NULL;
CREATE_PROCESS_DEBUG_INFO g_cpdi;
BYTE g_chINT3 = 0xCC, g_chOrgByte = 0;

BOOL OnCreateProcessDebugEvent(LPDEBUG_EVENT pde)
{
    // WriteFile() API 주소 구하기
    g_pfWriteFile = GetProcAddress(GetModuleHandleA("kernel32.dll"), "WriteFile");

    // API Hook - WriteFile()
    //   첫 번째 byte 를 0xCC (INT 3) 으로 변경 
    //   (orginal byte 는 백업)
    memcpy(&g_cpdi, &pde->u.CreateProcessInfo, sizeof(CREATE_PROCESS_DEBUG_INFO));
    ReadProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                      &g_chOrgByte, sizeof(BYTE), NULL);
    WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                       &g_chINT3, sizeof(BYTE), NULL);

    return TRUE;
}

BOOL OnExceptionDebugEvent(LPDEBUG_EVENT pde)
{
    CONTEXT ctx;
    PBYTE lpBuffer = NULL;
    DWORD dwNumOfBytesToWrite, dwAddrOfBuffer, i;
    PEXCEPTION_RECORD per = &pde->u.Exception.ExceptionRecord;

    // BreakPoint exception (INT 3) 인 경우
    if( EXCEPTION_BREAKPOINT == per->ExceptionCode )
    {
        // BP 주소가 WriteFile() 인 경우
        if( g_pfWriteFile == per->ExceptionAddress )
        {
            // #1. Unhook
            //   0xCC 로 덮어쓴 부분을 original byte 로 되돌림
            WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                               &g_chOrgByte, sizeof(BYTE), NULL);

            // #2. Thread Context 구하기
            ctx.ContextFlags = CONTEXT_CONTROL;
            GetThreadContext(g_cpdi.hThread, &ctx);

            // #3. WriteFile() 의 param 2, 3 값 구하기
            //   함수의 파라미터는 해당 프로세스의 스택에 존재함
            //   param 2 : ESP + 0x8
            //   param 3 : ESP + 0xC
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0x8), 
                              &dwAddrOfBuffer, sizeof(DWORD), NULL);
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0xC), 
                              &dwNumOfBytesToWrite, sizeof(DWORD), NULL);

            // #4. 임시 버퍼 할당
            lpBuffer = (PBYTE)malloc(dwNumOfBytesToWrite+1);
            memset(lpBuffer, 0, dwNumOfBytesToWrite+1);

            // #5. WriteFile() 의 버퍼를 임시 버퍼에 복사
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, 
                              lpBuffer, dwNumOfBytesToWrite, NULL);
            printf("\n### original string ###\n%s\n", lpBuffer);

            // #6. 소문자 -> 대문자 변환
            for( i = 0; i < dwNumOfBytesToWrite; i++ )
            {
                if( 0x61 <= lpBuffer[i] && lpBuffer[i] <= 0x7A )
                    lpBuffer[i] -= 0x20;
            }

            printf("\n### converted string ###\n%s\n", lpBuffer);

            // #7. 변환된 버퍼를 WriteFile() 버퍼로 복사
            WriteProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, 
                               lpBuffer, dwNumOfBytesToWrite, NULL);

            // #8. 임시 버퍼 해제
            free(lpBuffer);

            // #9. Thread Context 의 EIP 를 WriteFile() 시작으로 변경
            //   (현재는 WriteFile() + 1 만큼 지나왔음)
            ctx.Eip = (DWORD)g_pfWriteFile;
            SetThreadContext(g_cpdi.hThread, &ctx);

            // #10. Debuggee 프로세스를 진행시킴
            ContinueDebugEvent(pde->dwProcessId, pde->dwThreadId, DBG_CONTINUE);
            Sleep(0);

            // #11. API Hook
            WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                               &g_chINT3, sizeof(BYTE), NULL);

            return TRUE;
        }
    }

    return FALSE;
}

void DebugLoop()
{
    DEBUG_EVENT de;
    DWORD dwContinueStatus;

    // Debuggee 로부터 event 가 발생할 때까지 기다림
    while( WaitForDebugEvent(&de, INFINITE) )
    {
        dwContinueStatus = DBG_CONTINUE;

        // Debuggee 프로세스 생성 혹은 attach 이벤트
        if( CREATE_PROCESS_DEBUG_EVENT == de.dwDebugEventCode )
        {
            OnCreateProcessDebugEvent(&de);
        }
        // 예외 이벤트
        else if( EXCEPTION_DEBUG_EVENT == de.dwDebugEventCode )
        {
            if( OnExceptionDebugEvent(&de) )
                continue;
        }
        // Debuggee 프로세스 종료 이벤트
        else if( EXIT_PROCESS_DEBUG_EVENT == de.dwDebugEventCode )
        {
            // debuggee 종료 -> debugger 종료
            break;
        }

        // Debuggee 의 실행을 재개시킴
        ContinueDebugEvent(de.dwProcessId, de.dwThreadId, dwContinueStatus);
    }
}

int main(int argc, char* argv[])
{
    DWORD dwPID;

    if( argc != 2 )
    {
        printf("\nUSAGE : hookdbg.exe <pid>\n");
        return 1;
    }

    // Attach Process
    dwPID = atoi(argv[1]);
    if( !DebugActiveProcess(dwPID) )
    {
        printf("DebugActiveProcess(%d) failed!!!\n"
               "Error Code = %d\n", dwPID, GetLastError());
        return 1;
    }

    // 디버거 루프
    DebugLoop();

    return 0;
}
```

### 1. main()

```
int main(int argc, char* argv[])
{
    DWORD dwPID;

    if( argc != 2 )
    {
        printf("\nUSAGE : hookdbg.exe <pid>\n");
        return 1;
    }

    // Attach Process
    dwPID = atoi(argv[1]);
    if( !DebugActiveProcess(dwPID) )
    {
        printf("DebugActiveProcess(%d) failed!!!\n"
               "Error Code = %d\n", dwPID, GetLastError());
        return 1;
    }

    // 디버거 루프
    DebugLoop();

    return 0;
}
```

먼저 main() 함수를 보면 프로그램 실행 파라미터로 API 후킹하려는 프로세스의 PID를 받고 DebugActiveProcess()를 통해서 실행중인 프로세스에 Attach하여 디버깅을 시작합니다.

DebugActiveProcess() MSDN을 확인해보면 다음과 같이 입력한 PID를 파라미터로 넘겨줍니다.

![DebugActiveProcess() MSDN](https://user-images.githubusercontent.com/101708067/186913078-6704beab-8cd4-4c86-81e4-843602185fef.jpg)

그 다음 DebugLoop() 함수로 들어가서 디버기로부터 오는 디버그 이벤트를 처리합니다.

### 2. DebugLoop()

```
void DebugLoop()
{
    DEBUG_EVENT de;
    DWORD dwContinueStatus;

    // Debuggee 로부터 event 가 발생할 때까지 기다림
    while( WaitForDebugEvent(&de, INFINITE) )
    {
        dwContinueStatus = DBG_CONTINUE;

        // Debuggee 프로세스 생성 혹은 attach 이벤트
        if( CREATE_PROCESS_DEBUG_EVENT == de.dwDebugEventCode )
        {
            OnCreateProcessDebugEvent(&de);
        }
        // 예외 이벤트
        else if( EXCEPTION_DEBUG_EVENT == de.dwDebugEventCode )
        {
            if( OnExceptionDebugEvent(&de) )
                continue;
        }
        // Debuggee 프로세스 종료 이벤트
        else if( EXIT_PROCESS_DEBUG_EVENT == de.dwDebugEventCode )
        {
            // debuggee 종료 -> debugger 종료
            break;
        }

        // Debuggee 의 실행을 재개시킴
        ContinueDebugEvent(de.dwProcessId, de.dwThreadId, dwContinueStatus);
    }
}
```

DebugLoop() 함수는 윈도우 프로시저 함수(WndProc)와 유사하게 동작합니다. 이 함수는 디버기로부터 발생하는 이벤트를 받아서 처리한 후 디버기의 실행을 재개합니다.

이 함수에서 중요한 API는 **WaitForDebugEvent() API**입니다.

WaitForDebugEvent() MSDN을 확인해보면 디버기로 부터 디버그 이벤트가 발생할 때까지 기다리는 함수입니다. 

![WaitForDebugEvent() MSDN](https://user-images.githubusercontent.com/101708067/186913086-a2b48754-7ee4-42c7-8d32-7290b93fccee.jpg)

DebugLoop() 함수 코드에서 디버그 이벤트가 발생하면 WaitForDebugEvent() API는 첫 번째 파라미터인 de 변수(DEBUG_EVENT 구조체 객체)에 해당 이벤트에 대한 정보를 설정한 후 즉시 리턴합니다.

**DEBUG_EVENT 구조체**는 다음과 같습니다.

![DEBUG_EVENT구조체](https://user-images.githubusercontent.com/101708067/186913077-e6774252-e101-4b62-bf60-c3b751243c15.jpg)

디버그 이벤트는 9가지 종류가 있다고 했는데 DEBUG_EVENT.dwDebugEventCode 멤버에 9가지 이벤트 종류 중 하나가 세팅되며, 해당 이벤트 종류에 따라 적절한 DEBUG_EVENT.u(유니온) 멤버가 세팅됩니다.

**ContinueDebugEvent() API**는 디버기의 실행을 재개하는 함수입니다.

![ContinueDebugEvent() MSDN](https://user-images.githubusercontent.com/101708067/186913070-c469ba6d-beaa-4665-b519-2dd9ed7f56be.jpg)

ContinueDebugEvent() MSDN을 확인해보면 마지막 파라미터인 dwContinueStatus는 DBG_CONTINUE 또는 DBG_EXCEPTION_NOT_HANDLED 중에서 하나의 값을 가질 수 있습니다.

정상적으로 처리된 경우 DBG_CONTINUE로 세팅하고, 처리하지 못했거나 애플리케이션의 SEH(Structured Exception Handler)에서 처리하길 원할 때는 DBG_EXCEPTION_NOT_HANDLED로 세팅합니다.

DebugLoop() 함수에서는 세 가지 디버그 이벤트를 처리합니다.

1. EXIT_PROCESS_DEBUG_EVENT

디버기 프로세스가 종료될 때 발생하는 이벤트로 위 코드에서 이벤트가 발생하면 디버거도 같이 종료합니다.

2. CREATE_PROCESS_DEBUG_EVENT

CREATE_PROCESS_DEBUG_EVENT의 이벤트 핸들러인 OnCreateProcessDebugEvent()는 디버기의 프로세스가 시작(혹은 Attach)될 때 호출됩니다.

```
CREATE_PROCESS_DEBUG_INFO g_cpdi;

BOOL OnCreateProcessDebugEvent(LPDEBUG_EVENT pde)
{
    // WriteFile() API 주소 구하기
    g_pfWriteFile = GetProcAddress(GetModuleHandleA("kernel32.dll"), "WriteFile");

    // API Hook - WriteFile()
    //   첫 번째 byte 를 0xCC (INT 3) 으로 변경 
    //   (orginal byte 는 백업)
    memcpy(&g_cpdi, &pde->u.CreateProcessInfo, sizeof(CREATE_PROCESS_DEBUG_INFO));
    ReadProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                      &g_chOrgByte, sizeof(BYTE), NULL);
    WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                       &g_chINT3, sizeof(BYTE), NULL);

    return TRUE;
}
```

**OnCreateProcessDebugEvent()** 을 확인해보면 먼저 WriteFile() API의 시작 주소를 구합니다.

여기서 주목해야 하는 것은 디버기 프로세스의 메모리 주소가 아니라 디버거 프로세스의 메모리 주소를 얻어서 사용하는 것입니다. Windows OS에서 System DLL인 경우 모든 프로세스에서 동일한 주소(가상 메모리 주소)에 로딩되므로 이렇게 해도 문제 없습니다.

여기 나오는 g_cpdi는 전역변수로 선언하였고 **CREATE_PROCESS_DEBUG_INFO 구조체** 변수입니다.

![CREATE_PROCESS_DEBUG_INFO 구조체](https://user-images.githubusercontent.com/101708067/186913074-4e3eb142-a841-4ab6-bdef-ecabdd7679c7.jpg)

CREATE_PROCESS_DEBUG_INFO 구조체 hProcess 멤버(디버기 프로세스 핸들)를 이용하여 WriteFile() API를 후킹할 수 있습니다.

디버그 방법에서 후킹 방법은 API 시작 위치에 브레이크 포인트를 설치하면 됩니다. 디버기의 프로세스 핸들을 가지고 있기 때문에 ReadProcessMemory(), WriteProcessMemory() API를 이용하여 디버기의 프로세스 메모리 공간에 자유롭게 읽기/쓰기 작업을 할 수 있습니다.

### 3. EXCEPTION_DEBUG_EVENT - OnExceptionDebugEvent()

가장 핵심적인 EXCEPTION_DEBUG_EVENT 이벤트 핸들러인 OnExceptionDebugEvent()는 디버기의 INT3 명령을 처리하게 될 함수입니다.

```
BOOL OnExceptionDebugEvent(LPDEBUG_EVENT pde)
{
    CONTEXT ctx;
    PBYTE lpBuffer = NULL;
    DWORD dwNumOfBytesToWrite, dwAddrOfBuffer, i;
    PEXCEPTION_RECORD per = &pde->u.Exception.ExceptionRecord;

    // BreakPoint exception (INT 3) 인 경우
    if( EXCEPTION_BREAKPOINT == per->ExceptionCode )
    {
        // BP 주소가 WriteFile() 인 경우
        if( g_pfWriteFile == per->ExceptionAddress )
        {
            // #1. Unhook
            //   0xCC 로 덮어쓴 부분을 original byte 로 되돌림
            WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                               &g_chOrgByte, sizeof(BYTE), NULL);

            // #2. Thread Context 구하기
            ctx.ContextFlags = CONTEXT_CONTROL;
            GetThreadContext(g_cpdi.hThread, &ctx);

            // #3. WriteFile() 의 param 2, 3 값 구하기
            //   함수의 파라미터는 해당 프로세스의 스택에 존재함
            //   param 2 : ESP + 0x8
            //   param 3 : ESP + 0xC
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0x8), 
                              &dwAddrOfBuffer, sizeof(DWORD), NULL);
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0xC), 
                              &dwNumOfBytesToWrite, sizeof(DWORD), NULL);

            // #4. 임시 버퍼 할당
            lpBuffer = (PBYTE)malloc(dwNumOfBytesToWrite+1);
            memset(lpBuffer, 0, dwNumOfBytesToWrite+1);

            // #5. WriteFile() 의 버퍼를 임시 버퍼에 복사
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, 
                              lpBuffer, dwNumOfBytesToWrite, NULL);
            printf("\n### original string ###\n%s\n", lpBuffer);

            // #6. 소문자 -> 대문자 변환
            for( i = 0; i < dwNumOfBytesToWrite; i++ )
            {
                if( 0x61 <= lpBuffer[i] && lpBuffer[i] <= 0x7A )
                    lpBuffer[i] -= 0x20;
            }

            printf("\n### converted string ###\n%s\n", lpBuffer);

            // #7. 변환된 버퍼를 WriteFile() 버퍼로 복사
            WriteProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, 
                               lpBuffer, dwNumOfBytesToWrite, NULL);

            // #8. 임시 버퍼 해제
            free(lpBuffer);

            // #9. Thread Context 의 EIP 를 WriteFile() 시작으로 변경
            //   (현재는 WriteFile() + 1 만큼 지나왔음)
            ctx.Eip = (DWORD)g_pfWriteFile;
            SetThreadContext(g_cpdi.hThread, &ctx);

            // #10. Debuggee 프로세스를 진행시킴
            ContinueDebugEvent(pde->dwProcessId, pde->dwThreadId, DBG_CONTINUE);
            Sleep(0);

            // #11. API Hook
            WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
                               &g_chINT3, sizeof(BYTE), NULL);

            return TRUE;
        }
    }

    return FALSE;
}
```

코드를 보면 처음 if문에서 **EXCEPTION_BREAKPOINT 예외인지 체크**합니다. 그 다음 if문에서 **BP가 발생한 주소가 kernel32!WriteFile() 시작 주소와 같은지 체크**합니다.

WriteFile() 시작 주소는 OnCreateProcessDebugEvent()에서 미리 얻어 놓았으므로 이러한 두 가지 조건이 만족되면 아래의 1(Unhook) ~ 11(API Hook) 코드가 실행됩니다.

---

1. Unhook(API 훅 제거 )

```
//0xCC 로 덮어쓴 부분을 original byte 로 되돌림
WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
           &g_chOrgByte, sizeof(BYTE), NULL);
```

먼저 언훅을 하는 이유는 소문자가 대문자로 변경되는 작업 이후에 WriteFile()을 정상적인 상태로 호출하기 위해서입니다. 언훅 방법은 0xCC(INT3)로 덮어쓴 부분을 원래 바이트로 변경해주면 됩니다.

---

2. ThreadContext(쓰레드 컨텍스트) 구하기

모든 프로그램은 프로세스 단위로 실행되고 프로세스의 실제 명령어 코드는 쓰레드 단위로 실행됩니다. Windows OS는 멀티쓰레드 기반이기 때문에 하나의 프로세스에서 여러 쓰레드가 동시에 실행될 수 있습니다.

기존 쓰레드를 실행하면서 다음 실행에 필요한 중요한 정보는 CPU 레지스터 값 입니다. 그 쓰레드의 CPU 레지스터 정보를 저장하는 구조체가 CONTEXT 구조체 입니다.(쓰레드 하나당 CONTEXT 구조체 한 개)

```
ctx.ContextFlags = CONTEXT_CONTROL;
GetThreadContext(g_cpdi.hThread, &ctx);
```

위 코드에 GetThreadContext() API를 호출하면 ctx 구조체 변수에 해당 쓰레드의 CONTEXT를 저장하는 것을 확인할 수 있습니다.

---

3. WriteFile()의 param 2, 3 값 구하기

```
//   함수의 파라미터는 해당 프로세스의 스택에 존재함
//   param 2 : ESP + 0x8
//   param 3 : ESP + 0xC
ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0x8), 
            &dwAddrOfBuffer, sizeof(DWORD), NULL);
ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0xC), 
            &dwNumOfBytesToWrite, sizeof(DWORD), NULL);
```

WriteFile()를 호출할 때 넘어온 파라미터 중에서 param 2(쓰기버퍼 주소), param 3(버퍼 크기)를 알아내야 합니다. 함수의 파라미터는 스택에 저장되므로 CONTEXT.Esp 멤버를 이용해서 각각의 값을 구합니다.

---

    4 ~ 8. 소문자 → 대문자 변환 후 덮어쓰기

쓰기버퍼 주소와 크기를 구했으므로 이를 디버거 메모리 공간으로 읽어들인 후 소문자를 대문자로 변환합니다. 그리고 다시 원래 위치에 해당하는 디버기의 가상 메모리에 덮어쓰는 작업입니다.

```
// #4. 임시 버퍼 할당
lpBuffer = (PBYTE)malloc(dwNumOfBytesToWrite+1);
memset(lpBuffer, 0, dwNumOfBytesToWrite+1);

// #5. WriteFile() 의 버퍼를 임시 버퍼에 복사
ReadProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, 
            lpBuffer, dwNumOfBytesToWrite, NULL);
printf("\n### original string ###\n%s\n", lpBuffer);

// #6. 소문자 -> 대문자 변환
for( i = 0; i < dwNumOfBytesToWrite; i++ )
{
     if( 0x61 <= lpBuffer[i] && lpBuffer[i] <= 0x7A )
          lpBuffer[i] -= 0x20;
}

printf("\n### converted string ###\n%s\n", lpBuffer);

// #7. 변환된 버퍼를 WriteFile() 버퍼로 복사
WriteProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, 
lpBuffer, dwNumOfBytesToWrite, NULL);

// #8. 임시 버퍼 해제
free(lpBuffer);
```

---

9. Thread Context의 EIP를 WriteFile() 시작으로 변경하기

CONTEXT에서 Eip 멤버를 WriteFile() 시작 위치로 변경합니다.

EIP 현재 위치는 WriteFile() + 1(INT3)입니다.

```
//(현재는 WriteFile() + 1 만큼 지나왔음)
ctx.Eip = (DWORD)g_pfWriteFile;
SetThreadContext(g_cpdi.hThread, &ctx);
```

CONTEXT.Eip 멤버를 변경한 후 SetThreadContext() API를 호출합니다.

---

10. 디버거 프로세스 진행

이제 정상적인 WriteFile() API를 호출해야 하므로 ContinueDebugEvent() API를 호출하여 디버기 프로세스의 실행을 재개합니다.

```
ContinueDebugEvent(pde->dwProcessId, pde->dwThreadId, DBG_CONTINUE);
Sleep(0);
```

CONTEXT.Eip를 WriteFile() 시작으로 되돌렸으므로 WriteFile() 호출이 진행됩니다.

---

11. API 훅 설치

```
WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, 
          &g_chINT3, sizeof(BYTE), NULL);
```

다음에도 후킹을 하기 위해 API 훅을 설치합니다.

만약 API 훅을 설치하지 않으면 맨 처음 언훅되었기 때문에 WriteFile() API 후킹은 완전히 풀린상태가 돼 버립니다.

# 참고 문헌

[이승원] 리버싱 핵심원리 : 악성 코드 분석가의 리버싱 이야기

[reversecore · GitHub](https://github.com/reversecore) (리버싱 핵심원리 예제 GitHub)

https://reversecore.com/    (리버싱 핵심원리 저자 블로그)
