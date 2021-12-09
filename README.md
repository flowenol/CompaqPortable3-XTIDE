# Compaq Portable III XTIDE HOWTO

## Description of the problem

Optional ROM sockets are mapped at segment `0xe0000` while original
Compaq Portable III BIOS searches for optional ROMs between segments `0xc800`
and `0xdf80`. For this reason the two sockets cannot be fitted with any 27128
optional ROM chips.

The first word of the segment must be equal to `0xaa55` to be considered as the
beginning of optional ROM.

## Resources

POST codes for Compaq Portable at:
http://mrbios.com/techsupport/award/postcodes.htm

## POST Codes of interest:

* `28` - Perform search for optional ROMs (optional ROM detection)
* `1B` - The system ROM (performs BIOS ROM checksum)

## Basic methodology
Disassemble combined binary from odd and even images using software like
IDA v5.0 free version.
Look for code responsible for writing POST codes, which looks like this:

```
mov al,<POSTCODE>
out 84h,al
```

Compaq BIOS uses port `0x84` to write POST codes.

## Optional ROM address range extension
The BIOS code fragment which signals POST code `28` is to be found at offset
`0x3b59`, where subsequently subroutine at offset `0x7124` is called:

```
seg000:7124 ; ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦ S U B R O U T I N E ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦
seg000:7124
seg000:7124
seg000:7124 sub_F7124       proc near               ; CODE XREF: seg000:3B5Dp
seg000:7124                 mov     ax, 0C800h
seg000:7127                 cld
seg000:7128
seg000:7128 loc_F7128:                              ; CODE XREF: sub_F7124+Aj
seg000:7128                 call    sub_F717C
seg000:712B                 cmp     ax, 0DF80h
seg000:712E                 jbe     short loc_F7128
seg000:7130                 mov     ax, 0E000h
seg000:7133                 mov     ds, ax
seg000:7135                 assume ds:nothing
seg000:7135                 xor     bx, bx
seg000:7137                 cmp     word ptr [bx], 0AA55h
seg000:713B                 jnz     short loc_F7176
seg000:713D                 xor     si, si
seg000:713F                 mov     cx, 8000h
seg000:7142
seg000:7142 loc_F7142:                              ; CODE XREF: sub_F7124+23j
seg000:7142                 lodsw
seg000:7143                 add     bl, al
seg000:7145                 add     bl, ah
seg000:7147                 loop    loc_F7142
seg000:7149                 jz      short loc_F7163
seg000:714B                 mov     ax, 40h ; '@'
seg000:714E                 mov     ds, ax
seg000:7150                 assume ds:nothing
seg000:7150                 or      byte ptr ds:12h, 0FFh
seg000:7155                 mov     dx, 5000h
seg000:7158                 mov     bx, 0B57Fh
seg000:715B                 mov     cx, 0Fh
seg000:715E                 call    sub_F4771
seg000:7161                 jmp     short loc_F7176
seg000:7163 ; ------------------------------------------------------------------
seg000:7163
seg000:7163 loc_F7163:                              ; CODE XREF: sub_F7124+25j
seg000:7163                 mov     ax, 40h ; '@'
seg000:7166                 mov     es, ax
seg000:7168                 assume es:nothing
seg000:7168                 push    ds
seg000:7169                 mov     ds, ax
seg000:716B                 mov     ax, 3
seg000:716E                 push    ax
seg000:716F                 mov     bp, sp
seg000:7171                 call    dword ptr [bp+0]
seg000:7174                 pop     ax
seg000:7175                 pop     ax
seg000:7176
seg000:7176 loc_F7176:                              ; CODE XREF: sub_F7124+17j
seg000:7176                                         ; sub_F7124+3Dj
seg000:7176                 mov     ax, 40h ; '@'
seg000:7179                 mov     ds, ax
seg000:717B                 retn
seg000:717B sub_F7124       endp
seg000:717B
seg000:717C sub_F717C       proc near               ; CODE XREF: sub_F7124:loc_F7128p
seg000:717C                 mov     ds, ax
seg000:717E                 assume ds:nothing
seg000:717E                 xor     bx, bx
seg000:7180                 cmp     word ptr [bx], 0AA55h
seg000:7184                 jnz     short loc_F71E8
seg000:7186                 xor     si, si
seg000:7188                 xor     cx, cx
seg000:718A                 mov     ch, [bx+2]
seg000:718D                 shl     cx, 1
seg000:718F                 mov     dx, cx
seg000:7191
seg000:7191 loc_F7191:                              ; CODE XREF: sub_F717C+18j
seg000:7191                 lodsb
seg000:7192                 add     bl, al
seg000:7194                 loop    loc_F7191
seg000:7196                 jnz     short loc_F71D6
seg000:7198                 mov     cl, 4
seg000:719A                 shr     dx, cl
seg000:719C                 push    dx
seg000:719D                 mov     ax, 40h ; '@'
seg000:71A0                 mov     es, ax
seg000:71A2                 push    ds
seg000:71A3                 mov     ds, ax
seg000:71A5                 assume ds:nothing
seg000:71A5                 mov     ax, 3
seg000:71A8                 push    ax
seg000:71A9                 mov     si, sp
seg000:71AB                 push    bp
seg000:71AC                 xor     bp, bp
seg000:71AE                 call    dword ptr ss:[si]
seg000:71B1                 or      bp, bp
seg000:71B3                 jz      short loc_F71C1
seg000:71B5                 push    ds
seg000:71B6                 mov     ax, 40h ; '@'
seg000:71B9                 mov     ds, ax
seg000:71BB                 or      byte ptr ds:12h, 0FFh
seg000:71C0                 pop     ds
seg000:71C1                 assume ds:nothing
seg000:71C1
seg000:71C1 loc_F71C1:                              ; CODE XREF: sub_F717C+37j
seg000:71C1                 pop     bp
seg000:71C2                 pop     ax
seg000:71C3                 mov     ax, ds
seg000:71C5                 pop     ds
seg000:71C6                 pop     dx
seg000:71C7                 mov     bx, ds
seg000:71C9                 add     bx, dx
seg000:71CB                 cmp     ax, bx
seg000:71CD                 jbe     short loc_F71EB
seg000:71CF                 cmp     ax, 0DF80h
seg000:71D2                 jb      short locret_F71EF
seg000:71D4                 jmp     short loc_F71EB
seg000:71D6 ; ------------------------------------------------------------------
seg000:71D6
seg000:71D6 loc_F71D6:                              ; CODE XREF: sub_F717C+1Aj
seg000:71D6                 pusha
seg000:71D7                 push    es
seg000:71D8                 push    ds
seg000:71D9                 mov     dx, 5000h
seg000:71DC                 mov     bx, 0B6BCh
seg000:71DF                 mov     cx, 14h
seg000:71E2                 call    sub_F4771
seg000:71E5                 pop     ds
seg000:71E6                 pop     es
seg000:71E7                 assume es:nothing
seg000:71E7                 popa
seg000:71E8
seg000:71E8 loc_F71E8:                              ; CODE XREF: sub_F717C+8j
seg000:71E8                 mov     dx, 80h ; 'Ç'
seg000:71EB
seg000:71EB loc_F71EB:                              ; CODE XREF: sub_F717C+51j
seg000:71EB                                         ; sub_F717C+58j
seg000:71EB                 mov     ax, ds
seg000:71ED                 add     ax, dx
seg000:71EF
seg000:71EF locret_F71EF:                           ; CODE XREF: sub_F717C+56j
seg000:71EF                 retn
seg000:71EF sub_F717C       endp
```

The above routines are responsible for optional ROM discovery.
The range of search is between segments `0xc800` and `0xdf80`. It has to be
extended to `0xef80`. This can be done via modification of two instructions
at offsets `0x712b` and `0x71cf`:

Just modify two bytes:

`0x712d` from `0xdf` to `0xef`
`0x71d1` from `0xdf` to `0xef`

## Fixing the BIOS checksum

Now, once the routine has been modified, we must fix the BIOS checksum.
The routine which calculates the checksum is to be found when POST code `1B` is
signalled at offset `0x3ac3`. The soubroutine at offset `0x2094` is called:

```
seg000:2094 ; ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦ S U B R O U T I N E ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦
seg000:2094
seg000:2094
seg000:2094 sub_F2094       proc near               ; CODE XREF: seg000:3AC7p
seg000:2094                 push    ds
seg000:2095                 mov     ax, cs
seg000:2097                 mov     ds, ax
seg000:2099                 assume ds:nothing
seg000:2099                 mov     si, 8000h
seg000:209C                 xor     ah, ah
seg000:209E                 mov     cx, 8000h
seg000:20A1
seg000:20A1 loc_F20A1:                              ; CODE XREF: sub_F2094+10j
seg000:20A1                 lodsb
seg000:20A2                 add     ah, al
seg000:20A4                 loop    loc_F20A1
seg000:20A6                 jz      short loc_F20B6
seg000:20A8                 mov     dx, 5000h
seg000:20AB                 mov     bx, 0B57Fh
seg000:20AE                 mov     cx, 0Fh
seg000:20B1                 call    sub_F4771
seg000:20B4
seg000:20B4 loc_F20B4:                              ; CODE XREF: sub_F2094:loc_F20B4j
seg000:20B4                 jmp     short loc_F20B4
seg000:20B6 ; ------------------------------------------------------------------
seg000:20B6
seg000:20B6 loc_F20B6:                              ; CODE XREF: sub_F2094+12j
seg000:20B6                 pop     ds
seg000:20B7                 assume ds:nothing
seg000:20B7                 retn
seg000:20B7 sub_F2094       endp
```  

As can be deduced it just adds all ROM bytes and expects the sum to be `0`.

In order to achieve this we simply have to calculate the sum of all ROM bytes
except the last one, using something like HxD hex editor and then change the
last byte of ROM to value which added to the previously calculated sum will
produce `0`.

In our case the sum of all bytes except the last one is `0x57`, so we must add
`0xa9` for the result to be `0x00`. Change the last byte in ROM at offset `0x7fff`
to `0xa9`.

## Tools

Manipulate ROM images with srecord utilities.

Combine odd & even into single ROM:
```
srec_cat -o ROM.bin -binary even.bin -binary -unsplit 2 0 odd.bin -binary -unsplit 2 1
```

Split ROM into odd and even:
```
srec_cat -o changed_even.bin -binary changed.bin -binary -split 2 0
srec_cat -o changed_odd.bin -binary changed.bin -binary -split 2 1
```
