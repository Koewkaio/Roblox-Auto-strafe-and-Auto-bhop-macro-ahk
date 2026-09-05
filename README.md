# Roblox-Auto-strafe-and-Auto-bhop-macro-ahk
Dont need thank me later. 
You just need hold space and mouse your mouse, also in theses games:
https://www.roblox.com/games/5315066937/surf and https://www.roblox.com/games/5315046213/bhop
Oy6l and LainSURF uses this ahk

#Persistent
CoordMode, mouse, screen
global keyPressed := ""
x := 0
global prevX := 0
xDelta := 0
WinGetPos, x, y, width, height, Roblox
SysGet, numMonitors, MonitorCount
; Find the monitor that the window is on
monitorIndex := 1
Loop, %numMonitors%
{
    SysGet, monitor, MonitorWorkArea, %A_Index%
    if (x >= monitorLeft && x < monitorRight && y >= monitorTop && y < monitorBottom)
    {
        monitorIndex := A_Index
        break
    }
}
; Calculate the middle of the monitor that Roblox is on
SysGet, monitor, MonitorWorkArea, %monitorIndex%
global monitorMiddleX := (monitorRight + monitorLeft) // 2
mouseHook := SetWindowsHookEx(14, "LowLevelMouseProc")
SetWindowsHookEx(idHook, callback) {
    if (!hHook := DllCall("SetWindowsHookEx", "Int", idHook, "Ptr", RegisterCallback(callback, "Fast"), "Ptr", DllCall("GetModuleHandle", "UInt", 0, "Ptr"), "UInt", 0, "Ptr")) {
        throw (Exception(Format("0x{:X}", A_LastError), -1, FormatMessage(A_LastError)))
    }
    Static instance := {"__Class": "__HookEx"
            , "__Delete": Func("UnhookWindowsHookEx")}
    (hookEx := new instance()).Handle := hHook
    return (hookEx)
}
UnhookWindowsHookEx(hookEx) {
    if (!DllCall("UnhookWindowsHookEx", "Ptr", hookEx.Handle, "UInt")) {
        throw (Exception(Format("0x{:X}", A_LastError), -1, FormatMessage(A_LastError)))
    }
    return (True)
}
LowLevelMouseProc(nCode, wParam, lParam) {
    Critical, On
    if (!WinActive("Roblox"))
    {
        goto, returnhook
    }

    ;ToolTip, % rawX ", " screenMiddle
    if (wParam != 0x200)
    {
        goto, returnhook
    }

    MouseGetPos, rawX
    x := NumGet(lParam + 0, "Int")
    ; If this is the first mouse movement message, initialize prevX.
    if (prevX = 0)
    {
        prevX := %x%
        goto, returnhook
    }
    ; Calculate the distance the mouse has moved since the last message.
    xDelta := x - prevX
    ; Update prevX.
    prevX := x

    fullX := x + xDelta

    ; Check if mouse x is less than the middle of the screen or higher than the middle of the screen.
    if (!GetKeyState("Space", "P"))
    {
        keyPressed := ""
        goto, returnhook
    }
    ;ToolTip, % fullX
    if (fullX < monitorMiddleX)
    {
        if (keyPressed = "d")
        {
            SendInput, {d up}
            SendInput, {a down}
            keyPressed := "a"
        }
        else if (keyPressed = "")
        {
            SendInput, {a down}
            keyPressed := "a"
        }
    }
    else if (fullX > monitorMiddleX)
    {
        if (keyPressed = "a")
        {
            SendInput, {a up}
            SendInput, {d down}
            keyPressed := "d"
        }
        else if (keyPressed = "")
        {
            SendInput, {d down}
            keyPressed := "d"
        }
    }
    ;ToolTip, % x ", " keyPressed ", " xDelta
returnhook:
    return (DllCall("CallNextHookEx", "Ptr", 0, "Int", nCode, "UInt", wParam, "UInt", lParam))
}
; https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-formatmessage
FormatMessage(messageID) {
    Local
    if (!length := DllCall("Kernel32\FormatMessage", "UInt", 0x1100, "Ptr", 0, "UInt", messageID, "UInt", 0, "Ptr*", buffer := 0, "UInt", 0, "Ptr", 0, "UInt")) {
        return (FormatMessage(DllCall("Kernel32\GetLastError")))
    }
    return (StrGet(buffer, length - 2), DllCall("Kernel32\LocalFree", "Ptr", buffer, "Ptr"))  ;* Account for the newline and carriage return characters.
}
#IfWinActive Roblox
~$space up::
SendInput, {a up}
SendInput, {d up}
keyPressed := ""
return
; Register a hotkey to exit the script.
~F1::ExitApp
#IfWinActive
