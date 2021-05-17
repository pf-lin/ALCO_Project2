# ALCO_Project2
2-bit_Histroy_Predictor
## 專案說明
給定一個RISC-V的assembly code，並將所有branch instruction做prediction  

Input：一段RISC-V的assembly code，使用者可指定多少個entries  

Output：entry、目前做預測的branch instruction、predictor目前state和所有狀態、預測結果、實際結果、misprediction累積次數，以及列出所有entry之狀態  
## 程式流程
1.讀RISC-V assembly code(.txt檔)進來  

2.實際執行RISC-V code的程式  

3.每遇到branch instruction就做prediction  

4.根據每次的預測結果去修改該entry之predictor的狀態  

5.每預測一次就輸出一次，並show出所有entry之狀態  

6.直到RISC-V code執行結束  

## 程式碼解釋
`struct instruction` 包含 **rd** 、 **rs1** 、 **rs2** 和 **immediate**  

`struct predictor` 包含 **目前狀態** 、 **四個狀態** 和 **misprediction**  
## Sample Input
    0x110		li R2,0			; v=0 //addi R2,R0,0
    0x114		li R3,16		; Loop bound for LoopI //addi R3,R0,16
    0x118		li R4,0			; i=0 //addi R4,R0,0
            LoopI:
    0x11C		beq R4,R3,EndLoopI	; Exit LoopI if i==16  
    0x120		li R5,0			; j=0 //addi R5,R0,0  
            LoopJ:  
    0x124		beq R5,R3,EndLoopJ      ; Exit LoopJ if j==16  
    0x128		add R6,R5,R4		; j+i  
    0x12C		andi R6,R6,3		; (j+i)%4  
    0x130		bne R6,R0,Endif	        ; Skip if (j+i)%4!=0  
    0x134		add R2,R2,R5		; v+=j  
            Endif:  
    0x138		addi R5,R5,1		; j++  
    0x13C		beq R0,R0,LoopJ	        ; Go back to LoopJ  
            EndLoopJ:  
    0x140		addi R4,R4,1		; i++  
    0x144		beq R0,R0,LoopI	        ; Go back to LoopI  
            EndLoopI:  
## Sample Output
    Please input entry(entry > 0):  
    4  
    entry: 3                beq R4,R3,EndLoopI  
    (00, SN, SN, SN, SN) N N   misprediction: 0  
    all entries:  
    entry: 0 (00, SN, SN, SN, SN)  
    entry: 1 (00, SN, SN, SN, SN)  
    entry: 2 (00, SN, SN, SN, SN)  
    entry: 3 (00, SN, SN, SN, SN)  

    entry: 1                beq R5,R3,EndLoopJ  
    (00, SN, SN, SN, SN) N N   misprediction: 0  
    all entries:  
    entry: 0 (00, SN, SN, SN, SN)  
    entry: 1 (00, SN, SN, SN, SN)  
    entry: 2 (00, SN, SN, SN, SN)  
    entry: 3 (00, SN, SN, SN, SN)  

    entry: 0                bne R6,R0,Endif  
    (00, SN, SN, SN, SN) N N   misprediction: 0  
    all entries:  
    entry: 0 (00, SN, SN, SN, SN)  
    entry: 1 (00, SN, SN, SN, SN)  
    entry: 2 (00, SN, SN, SN, SN)  
    entry: 3 (00, SN, SN, SN, SN)  

    entry: 3                beq R0,R0,LoopJ  
    (00, SN, SN, SN, SN) N T   misprediction: 1  
    all entries:  
    entry: 0 (00, SN, SN, SN, SN)  
    entry: 1 (00, SN, SN, SN, SN)  
    entry: 2 (00, SN, SN, SN, SN)  
    entry: 3 (01, WN, SN, SN, SN)  

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (00, SN, SN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, SN, SN, SN)

    entry: 0                bne R6,R0,Endif
    (00, SN, SN, SN, SN) N T   misprediction: 1
    all entries:
    entry: 0 (01, WN, SN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, SN, SN, SN)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, SN, SN, SN) N T   misprediction: 2
    all entries:
    entry: 0 (01, WN, SN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, SN)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (01, WN, SN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, SN)

    entry: 0                bne R6,R0,Endif
    (01, WN, SN, SN, SN) N T   misprediction: 2
    all entries:
    entry: 0 (11, WN, WN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, SN)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, SN) N T   misprediction: 3
    all entries:
    entry: 0 (11, WN, WN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, WN)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, WN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, WN)

    entry: 0                bne R6,R0,Endif
    (11, WN, WN, SN, SN) N T   misprediction: 3
    all entries:
    entry: 0 (11, WN, WN, SN, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, WN)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, WN) N T   misprediction: 4
    all entries:
    entry: 0 (11, WN, WN, SN, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, WN, SN, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, WN, SN, WN) N N   misprediction: 3
    all entries:
    entry: 0 (10, WN, WN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, WT) T T   misprediction: 4
    all entries:
    entry: 0 (10, WN, WN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (10, WN, WN, SN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, WN, SN, SN) N T   misprediction: 4
    all entries:
    entry: 0 (01, WN, WN, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (01, WN, WN, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (01, WN, WN, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, WN, WN, SN) N T   misprediction: 5
    all entries:
    entry: 0 (11, WN, WT, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (11, WN, WT, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, WT, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, WT, WN, SN) N T   misprediction: 6
    all entries:
    entry: 0 (11, WN, WT, WN, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (11, WN, WT, WN, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, WT, WN, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, WT, WN, WN) N N   misprediction: 6
    all entries:
    entry: 0 (10, WN, WT, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (10, WN, WT, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (10, WN, WT, WN, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, WT, WN, SN) N T   misprediction: 7
    all entries:
    entry: 0 (01, WN, WT, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (01, WN, WT, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (01, WN, WT, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, WT, WT, SN) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, ST, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, WT, SN) N T   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, WT, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, WT, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, ST, WT, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, WT, WN) N N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (10, WN, ST, WT, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, WT, SN) T T   misprediction: 8
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N N   misprediction: 0
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WN, SN, ST) T T   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, SN, SN, SN) N T   misprediction: 1
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (01, WN, SN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, SN, SN, SN) N T   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WN, SN, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, WN, SN, ST) T N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, WN, SN, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, WN, SN, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, WN, SN, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, WN, SN, WT) N T   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, WN, WN, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, WN, WN, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, WN, WN, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, WN, WN, WT) N T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, WT) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N N   misprediction: 2
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, WT, WN, ST) T T   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WN, SN, SN) N T   misprediction: 3
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (01, WN, WN, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, WN, SN, SN) N T   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, WT, WN, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, WT, WN, ST) T N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, WT, WN, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (10, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, WT, WN, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, WT, WN, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, WT, WN, WT) N T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, WT, WT, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, WT, WT, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, WT, WT, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, WT, WT, WT) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, WT) T T   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 19
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 19
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N N   misprediction: 4
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, WT, ST) T T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, WT, SN, SN) N T   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (01, WN, WT, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, WT, SN, SN) T T   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, WT, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, WT, ST) T N   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, WT, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, WT, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, WT, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, WT, WT) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 22
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 22
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 23
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 23
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 23
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 23
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 24
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 24
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 24
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 5
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 24
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N N   misprediction: 24
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 24
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 24
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 25
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 25
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 25
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 25
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 26
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 26
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 26
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 26
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 27
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 27
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 27
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 27
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 6
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 28
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 29
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 29
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T T   misprediction: 29
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 30
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 30
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 30
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 30
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 31
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 31
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 31
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 31
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 32
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 32
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 32
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 32
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 7
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 33
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 33
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 33
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 34
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 34
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 34
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 35
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 36
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 36
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 36
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 37
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 38
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 38
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 38
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 39
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 40
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 8
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 40
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 40
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 40
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 40
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 40
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 41
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 41
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 41
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 41
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 42
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 42
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 42
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 42
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 43
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 43
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 43
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 9
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 43
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N N   misprediction: 43
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 43
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 43
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 44
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 44
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 44
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 44
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 45
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 45
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 45
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 45
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 46
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 46
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 46
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 46
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 10
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 47
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 48
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 48
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T T   misprediction: 48
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 49
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 49
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 49
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 49
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 50
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 50
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 50
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 50
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 51
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 51
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 51
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 51
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 11
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 52
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 52
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 52
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 53
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 53
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 53
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 54
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 55
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 55
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 55
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 56
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 57
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 57
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 57
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 58
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 59
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 12
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 59
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 18
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 59
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 59
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 18
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 59
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 18
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 59
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 60
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 60
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 60
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 60
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 61
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 61
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 61
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 61
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 62
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 62
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 62
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 13
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 62
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N N   misprediction: 62
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 19
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 62
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 19
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 62
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 63
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 63
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 63
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 63
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 64
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 64
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 64
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 64
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 65
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 65
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 65
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 65
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 14
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 66
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 19
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 67
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 67
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T T   misprediction: 67
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 68
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 68
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 68
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 68
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 69
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 69
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 69
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 69
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 70
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 70
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 70
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T T   misprediction: 70
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 15
    all entries:
    entry: 0 (11, WN, ST, ST, ST)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, ST) T N   misprediction: 71
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 20
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WT) T T   misprediction: 71
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WT) T T   misprediction: 71
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 72
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 72
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 72
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 73
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 74
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 74
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 74
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 75
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 76
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 76
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 76
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N T   misprediction: 77
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (11, WN, ST, ST, WT)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WT) T N   misprediction: 78
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 16
    all entries:
    entry: 0 (10, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, WN) T T   misprediction: 78
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 21
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T N   misprediction: 22
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (11, WN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, WN) T T   misprediction: 78
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (10, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (10, WN, ST, ST, WT) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (10, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (10, WN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 78
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (01, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (01, WN, ST, ST, WT) T T   misprediction: 22
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 1                beq R5,R3,EndLoopJ
    (00, WN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 78
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, WT)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, WT) T T   misprediction: 22
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 78
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 79
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 79
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 79
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 79
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 80
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 80
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 80
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 80
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, SN) N T   misprediction: 81
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (11, WN, ST, ST, WN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (11, WN, ST, ST, WN) N N   misprediction: 81
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (10, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (10, WN, ST, ST, SN) T T   misprediction: 81
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N N   misprediction: 17
    all entries:
    entry: 0 (01, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 0                bne R6,R0,Endif
    (01, WN, ST, ST, SN) T T   misprediction: 81
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R0,R0,LoopJ
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (00, SN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R5,R3,EndLoopJ
    (00, SN, ST, SN, SN) N T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (01, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 1                beq R0,R0,LoopI
    (01, WN, ST, SN, SN) T T   misprediction: 18
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)

    entry: 3                beq R4,R3,EndLoopI
    (11, WN, ST, ST, ST) T T   misprediction: 22
    all entries:
    entry: 0 (11, WN, ST, ST, SN)
    entry: 1 (11, WN, ST, SN, SN)
    entry: 2 (00, SN, SN, SN, SN)
    entry: 3 (11, WN, ST, ST, ST)
