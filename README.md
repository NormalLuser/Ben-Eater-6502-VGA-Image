# Ben-Eater-6502-VGA-Image
Display a image on a Ben Eater VGA+6502 by a mem copy from rom to ram
;Example Program for 6502 PC + VGA from https://eater.net/
  ;Fifty1Ford 1/1/2023
  ;To compile with Vasm:
  ;Get finch.bin from https://eater.net/downloads/finch.bin
  ;Delete the empty space at end of the file with something like
  ;Frhed - Free Hex Editor https://frhed.sourceforge.net/en/
  ;Delete everything to the end of the file starting at offset:
  ;9574 0x2566
  ;Save it as finchClip.bin. 
  ;It should now be about 9.4 KB instead of the 32KB of org
  ;Compile vasm6502_oldstyle -Fbin -dotdir -wdc02 VGAFinch.asm
  ;Thanks to: 
  ;https://github.com/DylanSpeiser/Java-6502-Emulator ;Emulator needed to debug
  ;https://www.lemon64.com/forum/viewtopic.php?t=69663 ;Got basic ram copy code here
  ;Good background info and example source code
  ;https://github.com/rehsd/VGA-6502 
  ;http://wilsonminesco.com/
  
  ;Lets put the clipped finch image data in the rom file.
  ;We'll put it at location 0x500 byte offset 1280 of the bin/rom
  ;this works out as $8500 or byte 34,048 in our 6502 system as
  ;the ROM chip is mapped to the upper half of the 64k address space 
  .org $8500 ;34,048
  incbin "C:\Projects\6502\finchClip.bin" ;Use the path you saved the file
  
  ;Normal org statement for our rom code to start at $8000/top 32 k
  .org $8000 ;32,768 

reset: ;start
Top:; loop location

;setup for image copy
 lda #$00 ;set our source memory address to copy from, $8500
 sta $FB
 lda #$85
 sta $FC
 lda #$00 ;set our destination memory to copy to, $2000, WRAM
 sta $FD
 lda #$20
 sta $FE
 ldy #$00 ;reset x and y for our loop
 ldx #$00

;lets not copy but reuse code to instead do a
;black screen fill first.
Loop:
 lda #%00000000 ;Load 'black' value into a
 sta ($FD),Y ;indirect index dest memory address, starting at $00
 INY
 bne Loop ;loop until our dest goes over 255
 ;cpy #$41 ;65 lines

 inc $FC ;increment high order source memory address, starting at $80
 inc $FE ;increment high order dest memory address, starting at $60
 lda $FE ;load high order mem address into a
 ;copy 68 lines
 cmp #$44;compare with the last address we want to write
 bne Loop ;if we're not there yet, loop

;Setup for image copy
 lda #$00 ;set our source memory address to copy from, $8500
 sta $FB
 lda #$85
 sta $FC
 lda #$00 ;set our destination memory to copy to, $2000, WRAM
 sta $FD
 lda #$20
 sta $FE
 ldy #$00 ;reset x and y for our loop
 ldx #$00

LoopI: ;Image loop
 lda ($FB),Y ;indirect index source memory address, starting at $00
 sta ($FD),Y ;indirect index dest memory address, starting at $00
 INY
 ;cpy #$41 ;65 lines
 bne LoopI ;loop until our dest goes over 255

 inc $FC ;increment high order source memory address, starting at $80
 inc $FE ;increment high order dest memory address, starting at $60
 lda $FE ;load high order mem address into a
 ;copy 68 lines
 cmp #$44 ;compare with the last address we want to write
 bne LoopI ;if we're not there yet, loop

;memory copy loop finished, 
;loop to location forever if you want to test speed
; jmp Top

Endless:
  jmp Endless

;Reset/IRQ/NMI always at last spots in ROM/address space
 .org $fffa ; 65,530 (65,536 is max)
 .word reset
 .word reset
 .word reset
