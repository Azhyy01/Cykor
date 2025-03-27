
/*  call_stack.
    
    실제 시스템에서는 스택이 메모리에 저장되지만, 본 과제에서는 `int` 배열을 이용하여 메모리를 구현합니다.
    원래는 SFP와 Return Address에 실제 가상 메모리 주소가 들어가겠지만, 이번 과제에서는 -1로 대체합니다.
    
    int call_stack[]      : 실제 데이터(`int 값`) 또는 `-1` (메타데이터 구분용)을 저장하는 int 배열 
    char stack_info[][]   : call_stack[]과 같은 위치(index)에 대한 설명을 저장하는 문자열 배열

    ==========================call_stack 저장 규칙==========================
    매개 변수 / 지역 변수를 push할 경우   : int 값 그대로
    Saved Frame Pointer 를 push할 경우  : call_stack에서의 index
    반환 주소값을 push할 경우       : -1
    =======================================================================
    

    ==========================stack_info 저장 규칙==========================
    매개 변수 / 지역 변수를 push할 경우        : 변수에 대한 설명
    Saved Frame Pointer 를 push할 경우  : 어떤 함수의 SFP인지
    반환 주소값을 push할 경우                 : "Return Address"
    ========================================================================
*/
#include <stdio.h>
#include <string.h>
#include <stdarg.h>
#define STACK_SIZE 50 // 최대 스택 크기

int     call_stack[STACK_SIZE];         // Call Stack을 저장하는 배열
char    stack_info[STACK_SIZE][20];     // Call Stack 요소에 대한 설명을 저장하는 배열

/*  SP (Stack Pointer), FP (Frame Pointer)

    SP는 현재 스택의 최상단 인덱스를 가리킵니다.
    스택이 비어있을 때 SP = -1, 하나가 쌓이면 `call_stack[0]` -> SP = 0, `call_stack[1]` -> SP = 1, ...

    FP는 현재 함수의 스택 프레임 포인터입니다.
    실행 중인 함수 스택 프레임의 sfp를 가리킵니다.
*/
int SP = -1; 
int FP = -1;

void func1(int arg1, int arg2, int arg3);
void func2(int arg1, int arg2);
void func3(int arg1);

void push(int value, char value_name[])
{
    if ((SP < (STACK_SIZE)-1))
    {
        ++SP;
        call_stack[SP] = value;
        strcpy(stack_info[SP], value_name);
        

    }

}
void pop()
{
    call_stack[SP] = 0;
    strcpy(stack_info[SP], "");
    SP--;

}

void Prologue(char* func_name, int count, ...)
{
    int bfFP = FP;

    int* bf_FP = &bfFP;//이전 FP
    va_list args;
    va_start(args, count); 
    for (int i = count-1; i >=0; i--) //함수 인자 stack에 push
    {
        int arg = va_arg(args, int);
        char temp[50];
        sprintf(temp, "arg%d", i+1);
        push(arg, temp);



    }
    va_end(args);
   
    push(-1, "return address",bf_FP);//return address push
    

    char sfp_info[50];
    sprintf(sfp_info, " %s SFP", func_name); //SFP push
    FP = SP;
    ++FP;
    push(*bf_FP, sfp_info);

}

void Epilogue(int count_local, int count_arg, int* SFP)
{
    for (int i = 0; i < count_local; i++)//지역 변수 pop
    {
        pop();
    }

    FP = *SFP; //SFP 복원
    pop();
    pop();//return address 제거
    for (int i = 0; i < count_arg; i++)
    {
        pop();
    }
}

/*  
    현재 call_stack 전체를 출력합니다.
    해당 함수의 출력 결과들을 바탕으로 구현 완성도를 평가할 예정입니다.
*/
void print_stack()
{
    if (SP == -1)        
    {
        printf("Stack is empty.\n");
        return;
    }

    printf("====== Current Call Stack ======\n");
    
    for (int i = SP; i >= 0; i--)  // 최상단에서 부터 내려감.
    {
        if (call_stack[i] != -1)    //return address가 아닌 경우
            printf("%d : %s = %d", i ,stack_info[i], call_stack[i]);
        else
            printf("%d : %s", i, stack_info[i]);   //return address인 경우

        if (i == SP)
            printf("    <=== [esp]\n");
        else if (i == FP)
            printf("    <=== [ebp]\n");
        else
            printf("\n");
    }
    printf("================================\n\n");
}


//func 내부는 자유롭게 추가해도 괜찮으나, 아래의 구조를 바꾸지는 마세요
void func1(int arg1, int arg2, int arg3)
{
    // func1의 스택 프레임 형성 (함수 프롤로그 + push)
    Prologue("func1", 3,arg3, arg2, arg1);

 
    int FP1 = FP; //복원할 func1의 SFP 저장
    int* func1_SFP = &FP1;
    int var_1 = 100;
    push(var_1, "var_1");

    
    print_stack();
   


    func2(11, 13);
    // func2의 스택 프레임 제거 (함수 에필로그 + pop)
    Epilogue(1, 2, func1_SFP);

    print_stack();
}


void func2(int arg1, int arg2)
{
    // func2의 스택 프레임 형성 (함수 프롤로그 + push)
    Prologue("func2", 2,arg2, arg1);
 
    int FP2 = FP; //복원할 func1의 SFP 저장
    int* func2_SFP = &FP2;
    int var_2 = 200;
    push(var_2, "var_2");
    
    
    print_stack();
    func3(77);
    // func3의 스택 프레임 제거 (함수 에필로그 + pop)
    Epilogue(2, 1, func2_SFP);
    print_stack();
}


void func3(int arg1)
{
    // func3의 스택 프레임 형성 (함수 프롤로그 + push)
    Prologue("func3", 1, arg1);
    
    int var_3 = 300;
    int var_4 = 400;
    push(var_3, "var_3");
    push(var_4, "var_4");


    
    print_stack();
}


//main 함수에 관련된 stack frame은 구현하지 않아도 됩니다.
int main()
{
    int* main_SFP = &FP;
    func1(1, 2, 3);
    
    // func1의 스택 프레임 제거 (함수 에필로그 + pop)
    Epilogue(1, 3, main_SFP);
    print_stack();
    return 0;
}

