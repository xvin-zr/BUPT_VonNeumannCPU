#include <stdio.h>
#include <string.h>
#include <math.h>
#include <windows.h>
//#include "operation.h"
#define PRE 8   // 前一地址开始的位数
#define LAT 12  // 后一地址开始的位数

//  @author: Xvin

int ax[9] = {0};
int ir = 0, ip = 0, flag = 0;
int code[128] = {0};
int data[256] = {0};
char ram[32768] = {'\0'};
//extern int ax[9];
//extern int ir, ip, flag;
// ip=计数器 每次+4
// flag= 0 / 1 / -1


int binaryToMode(const char *instruct);
int binaryToAddress(const char *instruct, int start);
int binaryToInum(const char *instruct);
int binaryToCode(const char *instruct);
int binaryToIR(const char *instruct);
void datatrans(int adres1, int adres2, int *data, int inum);
void calculate(int adres1, int adres2, int *data, int inum, int mode);
void logic(int adres1, int adres2, int *data, int inum, int mode);
void compare(int adres1, int adres2, const int *data, int inum);
void jump(int adres2, int inum);
void inout(int adres1, int *data, int mode);
void printSegment(int *code, int *data);
void printCur(void);
void analyse(const char *instruct, int *data, int mode);
void getRam(FILE *fp, char *ram, int *code);

int binaryToMode(const char *instruct) {
    int res = 0;
    for (int i = 0; i<8; i++) {
        res += (int)((instruct[i]-'0')*pow(2, 7-i));
    }
    return res;
}

int binaryToAddress(const char *instruct, int start) // start = 8 or 12
{
    int res = 0;
    for (int i = 0; i<4; i++) {
        res += (int)((instruct[start++]-'0')*pow(2, 3-i));
    }
    return res;
}

int binaryToInum(const char *instruct) {
    int res = 0;
    for (int i = 0; i<16; i++) {
        res += (int)((instruct[i+16]-'0')*pow(2, 15-i));
    }
    if (res>=32768){
        res -= 65536;
    }
    return res;
}

int binaryToCode(const char *instruct)
{
    int res=0;
    for ( int i=0; i<32; i++) {
        res += (int)((instruct[i]-'0')*pow(2, 31-i));
    }
    return res;
}

int binaryToIR(const char *instruct)
{
    int res=0;
    for ( int i=0; i<16; i++) {
        res += (int)((instruct[i]-'0')*pow(2, 15-i));
    }
    return res;
}

void datatrans(int adres1, int adres2, int *data, int inum) // 1
{
    if (adres2 == 0) {
        ax[adres1] = inum;
    } else if (adres1<=4 && adres2<=4) {
        ax[adres1] = ax[adres2];
    } else if (adres1>=5 && adres2>=5) {
        data[(ax[adres1]-16384)/2] = data[(ax[adres2]-16384)/2];
    } else if (adres1<=4 && adres2>=5) { // 将地址送至数据寄存器
        ax[adres1] = data[(ax[adres2]-16384)/2];
    } else if (adres1>=5 && adres2<=4) { // 将数据寄存器送至地址
        data[(ax[adres1]-16384)/2] = ax[adres2];
    }
}

void calculate(int adres1, int adres2, int *data, int inum, int mode) // 2, 3, 4, 5
{
    if (mode == 2) { //  +
        if (adres2 == 0) {
            if (adres1 <= 4) {
                ax[adres1] += inum;
            } else if (adres1 >=5) {
                data[(ax[adres1]-16384)/2] += inum;
            }
        } else if (adres1<=4 && adres2<=4) {
            ax[adres1] += ax[adres2];
        } else if (adres1>=5 && adres2>=5) {
            data[(ax[adres1]-16384)/2] += data[(ax[adres2]-16384)/2];
        } else if (adres1<=4 && adres2>=5) {
            ax[adres1] += data[(ax[adres2]-16384)/2];
        } else if (adres1>=5 && adres2<=4) {
            data[(ax[adres1]-16384)/2] += ax[adres2];
        }
    } else if (mode == 3) { //  -
        if (adres2 == 0) {
            if (adres1 <= 4) {
                ax[adres1] -= inum;
            } else if (adres1 >=5) {
                data[(ax[adres1]-16384)/2] -= inum;
            }
        }else if (adres1<=4 && adres2<=4) {
            ax[adres1] -= ax[adres2];
        } else if (adres1>=5 && adres2>=5) {
            data[(ax[adres1]-16384)/2] -= data[(ax[adres2]-16384)/2];
        } else if (adres1<=4 && adres2>=5) {
            ax[adres1] -= data[(ax[adres2]-16384)/2];
        } else if (adres1>=5 && adres2<=4) {
            data[(ax[adres1]-16384)/2] -= ax[adres2];
        }
    } else if (mode == 4) { //  *
        if (adres2 == 0) {
            if (adres1 <= 4) {
                ax[adres1] *= inum;
            } else if (adres1 >=5) {
                data[(ax[adres1]-16384)/2] *= inum;
            }
        }else if (adres1<=4 && adres2<=4) {
            ax[adres1] *= ax[adres2];
        } else if (adres1>=5 && adres2>=5) {
            data[(ax[adres1]-16384)/2] *= data[(ax[adres2]-16384)/2];
        } else if (adres1<=4 && adres2>=5) {
            ax[adres1] *= data[(ax[adres2]-16384)/2];
        } else if (adres1>=5 && adres2<=4) {
            data[(ax[adres1]-16384)/2] *= ax[adres2];
        }
    } else if (mode == 5) { //  /
        if (adres2 == 0) {
            if (adres1 <= 4) {
                ax[adres1] /= inum;
            } else if (adres1 >=5) {
                data[(ax[adres1]-16384)/2] /= inum;
            }
        }else if (adres1<=4 && adres2<=4) {
            ax[adres1] /= ax[adres2];
        } else if (adres1>=5 && adres2>=5) {
            data[(ax[adres1]-16384)/2] /= data[(ax[adres2]-16384)/2];
        } else if (adres1<=4 && adres2>=5) {
            ax[adres1] /= data[(ax[adres2]-16384)/2];
        } else if (adres1>=5 && adres2<=4) {
            data[(ax[adres1]-16384)/2] /= ax[adres2];
        }
    }
}

void logic(int adres1, int adres2, int *data, int inum, int mode) // 6, 7, 8    完成
{
    if (mode == 6) {
        if (adres1<=4 && adres2 == 0) {
            ax[adres1] = (ax[adres1] && inum);
        } else if (adres1>=5 && adres2 == 0) {
            data[(ax[adres1]-16384)/2] = (data[(ax[adres1]-16384)/2] && inum);
        } else if (adres1<=4 && adres2<=4) {
            ax[adres1] = (ax[adres1] && ax[adres2]);
        } else if (adres1>=5 && adres2>=5) {
            data[(ax[adres1]-16384)/2] = (data[(ax[adres1]-16384)/2] && data[(ax[adres2]-16384)/2]);
        } else if (adres1<=4 && adres2>=5) {
            ax[adres1] = (ax[adres1] && data[(ax[adres2]-16384)/2]);
        } else if (adres1>=5 && adres2<=4) {
            data[(ax[adres1]-16384)/2] = data[(ax[adres1]-16384)/2] && ax[adres2];
        }
    } else if (mode == 7) {
        if (adres1<=4 && adres2 == 0) {
            ax[adres1] = ax[adres1] || inum;
        } else if (adres1>=5 && adres2 == 0) {
            data[(ax[adres1]-16384)/2] = (data[(ax[adres1]-16384)/2] || inum);
        } else if (adres1<=4 && adres2<=4) {
            ax[adres1] = (ax[adres1] || ax[adres2]);
        } else if (adres1>=5 && adres2>=5) {
            data[(ax[adres1]-16384)/2] = (data[(ax[adres1]-16384)/2] || data[(ax[adres2]-16384)/2]);
        } else if (adres1<=4 && adres2>=5) {
            ax[adres1] = (ax[adres1] || data[(ax[adres2]-16384)/2]);
        } else if (adres1>=5 && adres2<=4) {
            data[(ax[adres1]-16384)/2] = data[(ax[adres1]-16384)/2] || ax[adres2];
        }
    } else if (mode == 8) {
        if (adres1<=4 && adres2 == 0) {
            ax[adres1] = !ax[adres1];
        } else if (adres1>=5 && adres2 == 0) {
            data[(ax[adres1]-16384)/2] = !data[(ax[adres1]-16384)/2];
        } else if (adres1 == 0 && adres2 <= 4) {
            ax[adres2] = !ax[adres2];
        } else if (adres1 == 0 && adres2 >= 5) {
            data[(ax[adres2]-16384)/2] = !data[(ax[adres2]-16384)/2];
        }
    }
}

void compare(int adres1, int adres2, const int *data, int inum) // 9
{
    if (adres1<=4 && adres2 == 0) {
        if (ax[adres1] > inum) {
            flag = 1;
        }  else if (ax[adres1] < inum) {
            flag = -1;
        } else {
            flag = 0;
        }
    } else if (adres1>=5 && adres2==0) {
        if (data[(ax[adres1]-16384)/2] > inum) {
            flag = 1;
        }  else if (data[(ax[adres1]-16384)/2] < inum) {
            flag = -1;
        } else {
            flag = 0;
        }
    } else if (adres1<=4 && adres2<=4) {
        if (ax[adres1] > ax[adres2]) {
            flag = 1;
        } else if (ax[adres1] < ax[adres2]) {
            flag = -1;
        } else {
            flag = 0;
        }
    } else if (adres1>=5 && adres2>=5) {
        if (data[(ax[adres1]-16384)/2] > data[(ax[adres2]-16384)/2]) {
            flag = 1;
        } else if (data[(ax[adres1]-16384)/2] < data[(ax[adres2]-16384)/2]) {
            flag = -1;
        } else {
            flag = 0;
        }
    } else if (adres1<=4 && adres2>=5) {
        if (ax[adres1] > data[(ax[adres2]-16384)/2]) {
            flag = 1;
        } else if (ax[adres1] < data[(ax[adres2]-16384)/2]) {
            flag = -1;
        } else {
            flag = 0;
        }
    } else if (adres1>=5 && adres2<=4) {
        if (data[(ax[adres1]-16384)/2] > ax[adres2]) {
            flag = 1;
        } else if (data[(ax[adres1]-16384)/2] < ax[adres2]) {
            flag = -1;
        } else {
            flag = 0;
        }
    }
}

void jump(int adres2, int inum) // 10
{
    if (adres2 == 0) {
        ip += inum-4;
    } else if (adres2 == 1) {
        if (flag == 0) {
            ip += inum-4;
        }
    } else if (adres2 == 2) {
        if (flag == 1) {
            ip += inum-4;
        }
    } else if (adres2 == 3) {
        if (flag == -1) {
            ip += inum-4;
        }
    }
}

void inout(int adres1, int *data, int mode) // 11, 12
{
    if (mode == 11) {
        printf("in:\n");
        if (adres1 <= 4) {
            scanf("%d", &ax[adres1]);
        } else if (adres1 >= 5) {
            scanf("%d", &data[(ax[adres1]-16384)/2]);
        }
    } else if (mode == 12) {
        printf("out: ");
        if (adres1 <= 4) {
            printf("%d\n", ax[adres1]);
        } else if (adres1 >= 5) {
            printf("%d\n", data[(ax[adres1]-16384)/2]);
        }
    }
}

void printSegment(int *code, int *data)
{
    int i;
    printf("\ncodeSegment :\n");
    for ( i=1; i<=8*16; i++) {
        if (i%8 != 0) {
            printf("%d ", code[i-1]);
        } else {
            printf("%d\n", code[i-1]);
        }
    }
    printf("\ndataSegment :\n");
    for ( i=1; i<=16*16; i++) {
        if (i%16 != 0) {
            printf("%d ", data[i-1]);
        } else {
            printf("%d\n", data[i-1]);
        }
    }
}

void printCur(void)
{
    printf("ip = %d\n"
           "flag = %d\n"
           "ir = %d\n",
           ip, flag, ir);
    for (int i=1; i<=8; ++i) {
        if (i%4 == 0) {
            printf("ax%d = %d\n", i, ax[i]);
        } else {
            printf("ax%d = %d ", i, ax[i]);
        }
    }
}

void analyse(const char *instruct, int *data, int mode)
{
    if (mode == 1) {
        datatrans(binaryToAddress(instruct, PRE), binaryToAddress(instruct, LAT),
                  data , binaryToInum(instruct));
    } else if (mode>=2 && mode<=5) {
        calculate(binaryToAddress(instruct, PRE), binaryToAddress(instruct, LAT), data,
                  binaryToInum(instruct), binaryToMode(instruct));
    } else if (mode>=6 && mode<=8) {
        logic(binaryToAddress(instruct, PRE), binaryToAddress(instruct, LAT), data,
              binaryToInum(instruct), binaryToMode(instruct));
    } else if (mode == 9) {
        compare(binaryToAddress(instruct, PRE), binaryToAddress(instruct, LAT), data,
                binaryToInum(instruct));
    } else if (mode == 10) {
        jump(binaryToAddress(instruct, LAT), binaryToInum(instruct));
    } else if (mode >= 11 && mode<=12) {
        inout(binaryToAddress(instruct, PRE), data, binaryToMode(instruct));
    }
}

void getRam(FILE *fp, char *ram, int *code)
{
    char c;
    char instruct[35] = {'\0'};
    int j=0;
    for (int i=0; ; i++){
        c = fgetc(fp);
        if (c == EOF) {
            break;
        } else {
            ram[i] = c;
        }
        if (ram[i] == '\n') {

            ram[i] = '\0';
            strcpy(instruct, ram + i - 32);
            code[j++] = binaryToCode(instruct);
        }
    }
//    puts(ram);
}

void runThread()
{
    char instruct[35] = {'\0'};

    FILE *fp = fopen("dict.dic", "r");
    if (fp) {
        getRam(fp, ram, code);
        while (1) {
            strcpy(instruct, ram+ip*8+ip/4); //   取指令
            if ( binaryToMode(instruct) ) {
//                code[ip/4] = binaryToCode(instruct);
                ip += 4;
                ir = binaryToIR(instruct);
                analyse(instruct, data, binaryToMode(instruct));
                printCur();
//                printSegment(code, data);
            } else {
                ip += 4;
                ir = binaryToIR(instruct);
                printCur();
                break;
            }
        }

        printSegment(code, data);

    }

    fclose(fp);
}


int main(void)
{
    runThread();

    return 0;
}


//  ax[] = data[ax[adres2]-16384]
