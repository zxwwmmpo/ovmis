```
format PE console 4.0
entry start

include 'win32a.inc'

section '.data' data readable writeable
    x dd 10
    y dd 3
    resAdd dd 0
    resSub dd 0
    resMul dd 0
    resDiv dd 0
	remainder dd 0

    filehandle dd ?
    bytesWritten dd ?

    formatStr db 'x = %d', 13, 10, 'y = %d', 13, 10, 'x + y = %d', 13, 10, 'x - y = %d', 13, 10, 'x * y = %d', 13, 10, 'x / y = %.3f', 13, 10, 0
    buffer db 256 dup(0)
    filename db 'END.txt', 0

    pauseMsg db 'Press any key to exit...', 0

section '.code' code readable executable
start:    
    mov eax, [x]
    add eax, [y]
    mov [resAdd], eax
    
    mov eax, [x]
    sub eax, [y]
    mov [resSub], eax

    mov eax, [x]
    imul eax, [y]
    mov [resMul], eax
    
    mov eax, [x]
    cdq   
    idiv dword [y]
    mov [resDiv], eax
	mov [remainder], edx

	fild dword [x]
	fidiv dword [y]
	fstp qword [resDiv]
    
    invoke sprintf, buffer, formatStr, [x], [y], [resAdd], [resSub], [resMul], dword [resDiv], dword [resDiv + 4]

    invoke CreateFile, filename, GENERIC_WRITE, 0, 0, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, 0    
    mov [filehandle], eax
    
    invoke lstrlen, buffer
    mov ebx, eax

        invoke WriteFile, [filehandle], buffer, ebx, bytesWritten, 0    
    
    invoke CloseHandle, [filehandle]

    invoke printf, pauseMsg    
    invoke getch
    
    invoke ExitProcess, 0

section '.idata' import data readable writeable
    library kernel32, 'kernel32.dll', \
            user32, 'user32.dll', \
            msvcrt, 'msvcrt.dll'

    import kernel32, \
           ExitProcess, 'ExitProcess', \
           CreateFile, 'CreateFileA', \
           WriteFile, 'WriteFile', \
           CloseHandle, 'CloseHandle', \
                   lstrlen, 'lstrlenA'

    import user32, \
           wsprintf, 'wsprintfA'

    import msvcrt, \
           printf, 'printf', \
           getch, '_getch', \
		   sprintf, 'sprintf'```
