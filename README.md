I made a maze game using 8086 Assmembly language and simulated it in emu8086
Here are the issues that needed to be fixed:
- Collision Detection: The check_collision procedure is not properly reverting the player's position when they attempt to move into a wall (#).
- Popup Messages: The wall collision message (wallMsg) and the congratulatory message (winMsg) are not being displayed correctly because the logic for handling these conditions is incomplete or incorrect.
- Player Movement Reversion: The player's position is not being reverted correctly when they attempt to move into a wall.

This is my first time making a game using assembly langauge and feel free to edit my code to fix the issues mentioned.

Below is the code of my maze game:

.model small
.stack 100h
.data
    instructions db "Maze Game Instructions:", 0Dh, 0Ah
                db "- Use W,A,S,D to move", 0Dh, 0Ah
                db "- Find the exit (E) to win", 0Dh, 0Ah
                db "- P is your character", 0Dh, 0Ah, 0Dh, 0Ah, '$'

    keyPrompt db "Enter move (W,A,S,D): ", '$'

    maze db "####################", 0Dh, 0Ah
         db "S                  #", 0Dh, 0Ah
         db "# # ############## #", 0Dh, 0Ah
         db "# #              # #", 0Dh, 0Ah
         db "# # ########### #  #", 0Dh, 0Ah
         db "### #         # #  #", 0Dh, 0Ah
         db "# # # ####### # #  #", 0Dh, 0Ah
         db "# #   #       #    #", 0Dh, 0Ah
         db "# # ### #########  #", 0Dh, 0Ah
         db "#                  E", 0Dh, 0Ah
         db "####################", '$'

    mazeWidth equ 20
    mazeHeight equ 11
    lineLength equ 22

    playerX db 1
    playerY db 1
    playerChar db 'P'
    lastMove db ?  ; Track last movement direction (U=up, D=down, L=left, R=right)

    wallMsg db 0Dh, 0Ah, "You can't walk through walls! NO CHEATING!!!", 0Dh, 0Ah, '$'
    winMsg db 0Dh, 0Ah, "Congratulations! You found the exit!", 0Dh, 0Ah, '$'
    exitGame db 0Dh, 0Ah, "Press any key to exit...", '$'

.code
main proc
    mov ax, @data
    mov ds, ax

    ; Set video mode to text 80x25
    mov ax, 3
    int 10h

game_loop:
    call display_maze

    ; Display instructions
    mov ah, 9
    lea dx, instructions
    int 21h

    ; Display key prompt
    mov ah, 9
    lea dx, keyPrompt
    int 21h

    ; Get user input
    mov ah, 1
    int 21h

    ; Convert to uppercase
    and al, 0DFh

    ; Check which key was pressed
    cmp al, 'W'
    je move_up
    cmp al, 'A'
    je move_left
    cmp al, 'S'
    je move_down
    cmp al, 'D'
    je move_right
    jmp game_loop  ; Invalid input - loop again

move_up:
    mov lastMove, 'U'
    dec playerY
    call check_collision
    jmp game_loop

move_left:
    mov lastMove, 'L'
    dec playerX
    call check_collision
    jmp game_loop

move_down:
    mov lastMove, 'D'
    inc playerY
    call check_collision
    jmp game_loop

move_right:
    mov lastMove, 'R'
    inc playerX
    call check_collision
    jmp game_loop

check_collision:
    ; Check if player is out of bounds
    cmp playerX, 0
    jl undo_move
    cmp playerY, 0
    jl undo_move
    cmp playerX, mazeWidth-1
    jg undo_move
    cmp playerY, mazeHeight-1
    jg undo_move
    
    ; Get the character at the new position
    call get_maze_char
    
    ; Check for wall collision
    cmp al, '#'
    je wall_collision
    
    ; Check for win condition
    cmp al, 'E'
    je win_game
    
    ret

wall_collision:
    ; Display wall collision message
    mov ah, 9
    lea dx, wallMsg
    int 21h
    
undo_move:
    ; Determine which move to undo based on lastMove
    cmp lastMove, 'U'
    je undo_move_up
    cmp lastMove, 'D'
    je undo_move_down
    cmp lastMove, 'L'
    je undo_move_left
    cmp lastMove, 'R'
    je undo_move_right
    ret

undo_move_up:
    inc playerY
    ret
undo_move_down:
    dec playerY
    ret
undo_move_left:
    inc playerX
    ret
undo_move_right:
    dec playerX
    ret

win_game:
    ; Clear screen
    mov ax, 3
    int 10h
    
    ; Display the maze one last time
    call display_maze
    
    ; Display win message
    mov ah, 9
    lea dx, winMsg
    int 21h
    
    ; Display exit prompt
    mov ah, 9
    lea dx, exitGame
    int 21h
    
    ; Wait for any key
    mov ah, 1
    int 21h
    
    ; Exit program
    mov ax, 4C00h
    int 21h

display_maze proc
    ; Clear screen
    mov ax, 3
    int 10h
    
    ; Display the maze
    mov ah, 9
    lea dx, maze
    int 21h

    ; Calculate cursor position for player
    ; Row = playerY + 1 (to skip the first line which is instructions)
    mov dh, playerY
    inc dh
    mov dl, playerX
    
    ; Set cursor position
    mov ah, 2
    mov bh, 0
    int 10h

    ; Display player character
    mov ah, 0Eh
    mov al, playerChar
    int 10h
    
    ret
display_maze endp

get_maze_char proc
    ; Input: playerX and playerY are the coordinates
    ; Output: AL contains the character at that position
    
    ; Calculate offset in maze
    ; Each line is 22 bytes (20 chars + CR + LF)
    mov al, playerY
    mov bl, lineLength
    mul bl      ; AX = playerY * lineLength
    
    ; Add playerX
    mov bl, playerX
    mov bh, 0
    add ax, bx  ; AX now has the offset
    
    ; Get the character from maze
    lea si, maze
    add si, ax
    mov al, [si]
    
    ret
get_maze_char endp

end main
