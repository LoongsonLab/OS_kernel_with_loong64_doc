# CSR寄存器


# CSR指令

## CSRRD指令

## CSRWR指令

## CSRXCHG指令

```
1.  csrrd  a0, NUM
2.  andi   a0, 0x111
3.  csrwr  a0, NUM

	

```

1. 清空CSR Reg的某一位
```
li.d a0, 0x00001
csrxchg 

```






# IOCSR指令

## IOCSRRD指令

iocsr 

ld.d a0, 0x111111



## IOCSRWR指令



