# school
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#pragma pack(1)
const char *DataFilename = "d:\\classfee.data";
typedef struct classfee ClassFee;
struct classfee{
    int id;         /*收支编号*/
    char io;
	char time[256];    /*日期*/
    char cbr[10];       /*经办人*/
    char reason[256];      /*原因*/
    float fee;       /*金额*/
    float restfee;         /*余额*/
    ClassFee *next;  
};
 
void ShowMenu() {
const char *menu = {
    "1.新增收入	\n"
    "2.新增支出 \n"
	"3.班费明细\n"
    "4.退出\n 请选择:（1-4）\n"
    };
    printf("%s", menu);
}
 
ClassFee *fee;
int index = 0;
float lastfee = 0.0f;
 
int InitDataFile() {
    FILE *fp = fopen(DataFilename, "rb");  /*打开文件*/
 
    size_t sizefee = sizeof(ClassFee);   /*分配文件大小*/
    fee = (ClassFee*)malloc(sizefee);    /*指针长度+1*/
    fee->next = NULL;
 
    if (fp==NULL) {
        return 1;
    }
 
    ClassFee *p = fee;
    size_t len;
 
    while (!feof(fp)) {
        ClassFee *q = (ClassFee*)malloc(sizefee);  /*指针长度+1*/
        len = fread((char*)q, sizefee, 1, fp);
        if (len==1) {
            index++;
            lastfee = q->restfee;     /*最后一次取得数据为最后余额*/
            q->next = NULL;
            p->next = q;
            p = q;
        }
    }
 
    fclose(fp);
    return 0;
}
 
int WriteDataFile() {
    FILE *fp = fopen(DataFilename, "wb");
 
    size_t sizefee = sizeof(ClassFee);
    ClassFee *p = fee->next;
 
    while (p) {
        fwrite((char *)p, sizefee, 1, fp);
        p = p->next;
    }
 
    fclose(fp);
    return 0;
}
 
void FreeResource() {
    ClassFee *p = fee, *q=NULL;
    while (p) {
        q = p->next;
        free(p);
        p=q;
    }
}
 
void AppendFee(ClassFee *f) {
    ClassFee *p = fee, *q;
    q = p->next;
    while (q) {
        p=q;
        q=p->next;
    }
    p->next = f;
}
 
void income() {                      /*班费收入程序*/
    ClassFee *p = (ClassFee*)malloc(sizeof(ClassFee));
    p->id = ++index;
    fflush(stdin);
    printf("输入支出费用信息：\n");
    p->io = 1;
    printf("  日期:");
	scanf("%s", p->time);
    printf("  经办人:");
	scanf("%s", p->cbr);
    printf("  原因:");
	scanf("%s", p->reason);
    printf("  金额:");
	scanf("%f", &p->fee);
    p->restfee = (lastfee+p->fee);           /*金额加上收入的金额取得数据为最后余额*/
    lastfee = p->restfee;
    p->next = NULL;
    AppendFee(p);
    printf("-------------------------------------------------\n");
}
	
	void spending() {         /*班费支出程序*/
    ClassFee *p = (ClassFee*)malloc(sizeof(ClassFee));
    p->id = ++index;
    fflush(stdin);
    printf("输入支出费用信息:\n");
	printf("  日期:");
	scanf("%s", p->time);
    printf("  经办人:");
	scanf("%s", p->cbr);
    printf("  原因:");
	scanf("%s", p->reason);
    printf("  金额:");
	scanf("%f", &p->fee);
    p->restfee=(lastfee-p->fee);      /*金额减去支出的金额取得数据为最后余额*/
    lastfee = p->restfee;
    p->next = NULL;
    AppendFee(p);
    printf("-------------------------------------------------\n");
}

	void DisplayFee(ClassFee *p) {                /*班费明细程序*/
    printf("  收支编号 :   %d\n", p->id);
    printf("  收入/支出:   %s\n", p->io == 1?"收入":"支出");
	printf("  日期     :   %s\n", p->time);
    printf("  经办人   :   %s\n", p->cbr);
    printf("  原因     :   %s\n", p->reason);
    printf("  金额     :   %.2f\n", p->fee);
    printf("  余额     :   %.2f\n", p->restfee);
    printf("-------------------------------------------------\n");
}

 
int main() {
    int choice;
 
    InitDataFile();
printf("index=%d, lastfee=%.2f\n", index, lastfee);
    while (1) {
        ShowMenu();
        scanf("%d", &choice);
        if (choice<1 || choice>5) {         /*当输入的数据不在范围1-4显示输入错误*/
            system("cls");
            printf("你的输入错误，请重新输入\n------------------------\n");
            continue;
		} else {
            if (choice==4) {             
                WriteDataFile();
                break;
            } else if (choice==1) {
                income();
			} else if (choice==2) {
                spending();
            } else if (choice==3) {
                system("cls");
                ClassFee *p = fee->next;
                while (p) {
                    DisplayFee(p);
                    p=p->next;
                }
            
            }
        }
    }
    return 0;
}
