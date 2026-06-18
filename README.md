#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <windows.h>
#include <conio.h>

#define MAX_RANK 10

typedef struct {
    char name[20];
    int score;
} Rank;

void clearScreen() {
    system("cls");
}

void wait(int ms) {
    Sleep(ms);
}

int inputNumber() {
    char line[100];
    int num;

    while (1) {
        fgets(line, sizeof(line), stdin);
        if (sscanf(line, "%d", &num) == 1) {
            return num;
        }
        printf("  >> 숫자로 다시 입력하세요: ");
    }
}

void flushInput() {
    while (_kbhit()) _getch();
}

void printTitle() {
    printf("\n");
    printf("  ##########################################################\n");
    printf("  #                                                        #\n");
    printf("  #          [*]  등 교  생 존  게 임  [*]                 #\n");
    printf("  #                                                        #\n");
    printf("  #         실력으로 지각을 막아라! (운+실력 게임)         #\n");
    printf("  #                                                        #\n");
    printf("  ##########################################################\n");
    printf("\n");
}

void printBar(int value, int maxValue, int length) {
    int filled, i;
    if (value < 0) value = 0;
    filled = (value * length) / maxValue;
    if (filled > length) filled = length;

    printf("[");
    for (i = 0; i < length; i++) {
        if (i < filled) printf("#");
        else printf("-");
    }
    printf("]");
}

void printStatus(int hp, int money, int time, int combo) {
    printf("  +------------------------------------------------------+\n");
    printf("  |  [ 현재 상태 ]                                       \n");
    printf("  |                                                      \n");
    printf("  |  체력  ");
    printBar(hp, 120, 20);
    printf(" %4d        \n", hp);
    printf("  |  돈    ");
    printBar(money, 12000, 20);
    printf(" %5d원     \n", money);
    printf("  |  시간  경과 %3d분                                    \n", time);
    printf("  |  콤보  %d연속 성공 (보너스 점수!)                    \n", combo);
    printf("  +------------------------------------------------------+\n");
}

void printItem(int drink, int coupon, int umbrella) {
    printf("  [ 보유 아이템 ]  ");
    printf("에너지드링크 x%d   ", drink);
    printf("할인권 x%d   ", coupon);
    printf("우산 x%d\n", umbrella);
}

/* 환승 타이밍 게임: 목표 구간 고정, > 표시가 움직임 */
int timingGame(const char* title, int difficulty) {
    int barLen = 55;
    int targetStart, targetEnd;
    int pos = 0;
    int dir = 1;
    int speed;
    int targetWidth;
    int i, k;
    int pressed = 0;
    int success = 0;
    char bar[100];

    /* 난이도별: 목표 폭이 좁아지고 커서가 빨라짐 */
    if (difficulty == 1) { targetWidth = 9; speed = 60; }
    else if (difficulty == 2) { targetWidth = 7; speed = 45; }
    else { targetWidth = 4; speed = 32; }

    targetStart = 8 + rand() % (barLen - targetWidth - 16);
    targetEnd = targetStart + targetWidth - 1;

    printf("\n  >> %s\n", title);
    printf("  >> [=====] 구간은 고정입니다.\n");
    printf("  >> 움직이는 > 표시가 목표 구간에 왔을 때 [스페이스] 또는 [엔터]!\n\n");
    wait(900);

    flushInput();

    for (i = 0; i < barLen * 2 * 4; i++) {
        for (k = 0; k < barLen; k++) {
            if (k == pos) bar[k] = '>';
            else if (k >= targetStart && k <= targetEnd) bar[k] = '=';
            else bar[k] = '-';
        }
        bar[barLen] = '\0';

        printf("\r  |%s|   ", bar);
        fflush(stdout);

        if (_kbhit()) {
            int c = _getch();
            if (c == ' ' || c == '\r' || c == 13) {
                pressed = 1;
                if (pos >= targetStart && pos <= targetEnd) success = 1;
                break;
            }
        }

        pos += dir;

        if (pos >= barLen - 1) {
            pos = barLen - 1;
            dir = -1;
        }
        else if (pos <= 0) {
            pos = 0;
            dir = 1;
        }

        wait(speed);
    }

    printf("\n");

    if (!pressed) {
        printf("  >> 시간 초과! 타이밍을 놓쳤습니다.\n");
        return 0;
    }

    if (success) {
        printf("  >> 성공! 완벽한 타이밍!\n");
        return 1;
    }
    else {
        printf("  >> 실패! 빗나갔습니다.\n");
        return 0;
    }
}

/* 도보 기본 미니게임: 장애물 피하기 */
int jumpGame(int difficulty) {
    int track = 40;
    int charPos = 5;
    int obstacleCount;
    int speed;
    int jumpRange;
    int cleared = 0;
    int n, i;

    /* 난이도별: 장애물 수는 4개로 동일, 난이도가 오를수록 속도만 빨라짐
       (점프 인정 범위도 살짝 좁혀 체감 난이도 부여) */
    obstacleCount = 4;
    if (difficulty == 1) { speed = 95; jumpRange = 5; }
    else if (difficulty == 2) { speed = 65; jumpRange = 4; }
    else { speed = 40; jumpRange = 3; }

    printf("\n  >> 장애물이 다가옵니다! 가까워지면 [스페이스]로 점프!\n");
    printf("  >> 총 %d개의 장애물을 넘어야 합니다.\n\n", obstacleCount);
    wait(1200);

    flushInput();

    for (n = 0; n < obstacleCount; n++) {
        int obsPos = track - 1;
        int jumpFrame = 0;
        int success = 0;

        while (obsPos >= 0) {
            if (_kbhit()) {
                int c = _getch();

                if (c == ' ' || c == '\r' || c == 13) {
                    jumpFrame = 8;

                    if (obsPos - charPos <= jumpRange && obsPos - charPos >= -1) {
                        success = 1;
                    }
                }
            }

            clearScreen();
            printf("\n  [ 도보 기본 미션: 장애물 피하기 ]\n");
            printf("  >> [스페이스]로 점프!   (넘음 %d/%d)\n\n", cleared, obstacleCount);

            printf("  |");
            for (i = 0; i < track; i++) {
                if (jumpFrame > 0 && i == charPos) printf("O");
                else printf(" ");
            }
            printf("|\n");

            printf("  |");
            for (i = 0; i < track; i++) {
                if (jumpFrame <= 0 && i == charPos) printf("O");
                else if (i == obsPos) printf("#");
                else printf("_");
            }
            printf("|\n");

            if (jumpFrame > 0) {
                printf("\n        점프 중!   \\(^o^)/\n");
                jumpFrame--;
            }

            if (obsPos == charPos && !success) {
                printf("\n  >> 쿵! 장애물에 부딪혔습니다!\n");
                return cleared;
            }

            obsPos--;
            wait(speed);
        }

        if (success) cleared++;
        flushInput();
    }

    printf("\n  >> 모든 장애물을 넘었습니다!\n");
    return cleared;
}

/* 자전거 기본 미니게임: 제한시간 안에 스페이스 연타
   - 화면을 매 프레임 지우지 않고 \r 로 같은 줄만 갱신 -> 입력 안정 + 깜빡임 제거
   - 키 반복 자동입력 영향을 줄이기 위해 한 프레임에 1회만 카운트 */
int pedalGame(int difficulty) {
    int timeLimit;
    int count = 0;
    int target;
    int elapsed = 0;
    int i;

    /* 난이도별: 목표 횟수 증가, 제한시간 감소 */
    if (difficulty == 1) { target = 15; timeLimit = 4000; }
    else if (difficulty == 2) { target = 22; timeLimit = 3500; }
    else { target = 30; timeLimit = 3000; }

    clearScreen();
    printf("\n  [ 자전거 기본 미션: 페달 밟기 게임 ]\n\n");
    printf("  >> 제한시간 안에 [스페이스]를 최대한 많이 누르세요!\n");
    printf("  >> 목표 횟수: %d번\n\n", target);
    printf("  3..\n"); wait(500);
    printf("  2..\n"); wait(500);
    printf("  1..\n"); wait(500);
    printf("  시작!\n\n");

    flushInput();

    while (elapsed < timeLimit) {
        /* 이번 프레임에 들어온 키를 모두 처리하되, 한 번 누름은 한 번만 카운트 */
        int pressedThisFrame = 0;
        while (_kbhit()) {
            int c = _getch();
            if (c == ' ' && !pressedThisFrame) {
                count++;
                pressedThisFrame = 1;
            }
        }

        /* 진행 막대 (같은 줄 갱신) */
        clearScreen();

        printf("\n");
        printf("      __o\n");
        printf("    _ \\<,_\n");
        printf("   (_)/ (_)\n\n");
        printf("\r  페달 [");
        for (i = 0; i < 30; i++) {
            if (i < (count * 30) / target) printf("#");
            else printf("-");
        }
        printf("] %2d/%d  남은시간 %.1f초   ", count, target, (timeLimit - elapsed) / 1000.0);
        fflush(stdout);

        wait(60);
        elapsed += 60;
    }

    printf("\n\n  >> 최종 페달 횟수: %d번\n", count);

    if (count >= target) {
        printf("  >> 성공! 빠르게 페달을 밟았습니다!\n");
        return 1;
    }
    else {
        printf("  >> 실패! 페달 속도가 부족했습니다!\n");
        return 0;
    }
}

void moveAnimation(int choice) {
    char vehicle[20];
    int i, j;

    if (choice == 1) strcpy(vehicle, "[지하철]");
    else if (choice == 2) strcpy(vehicle, "[버 스]");
    else if (choice == 3) strcpy(vehicle, "<<자전거");
    else strcpy(vehicle, "(o_o)/");

    printf("\n");
    for (i = 0; i < 30; i += 5) {
        printf("\r  ");
        for (j = 0; j < i; j++) printf(" ");
        printf("%s  >>>", vehicle);
        fflush(stdout);
        wait(150);
    }

    printf("\r  ");
    for (j = 0; j < 30; j++) printf(" ");
    printf("%s  >>> 도착!\n", vehicle);
    wait(400);
}

void randomItem(int* drink, int* coupon, int* umbrella) {
    int r = rand() % 100;

    if (r < 20) {
        printf("\n  >> 아이템 발견! [에너지 드링크] 를 얻었습니다.\n");
        (*drink)++;
    }
    else if (r < 35) {
        printf("\n  >> 아이템 발견! [교통비 할인권] 을 얻었습니다.\n");
        (*coupon)++;
    }
    else if (r < 45) {
        printf("\n  >> 아이템 발견! [우산] 을 얻었습니다.\n");
        (*umbrella)++;
    }
}

/* 지하철 기본 미니게임 */
void subwayTransfer(int* time, int difficulty, int* combo) {
    int ok;

    printf("\n");
    printf("  ------------------------------------------\n");
    printf("  *  지하철 기본 미션: 환승 타이밍 맞추기!\n");
    printf("  ------------------------------------------\n");
    wait(500);

    ok = timingGame("지하철 환승 타이밍 맞추기", difficulty);

    if (ok) {
        printf("  >> 환승 성공! 시간 손실 없음. (콤보 +1)\n");
        (*combo)++;
    }
    else {
        printf("  >> 환승 실패! 시간 +20분. (콤보 초기화)\n");
        *time += 20;
        *combo = 0;
    }
    wait(700);
}

/* 버스 기본 미니게임 */
void busTransfer(int* time, int difficulty, int* combo) {
    int ok;

    printf("\n");
    printf("  ------------------------------------------\n");
    printf("  *  버스 기본 미션: 환승 타이밍 맞추기!\n");
    printf("  ------------------------------------------\n");
    wait(500);

    ok = timingGame("버스 환승 타이밍 맞추기", difficulty);

    if (ok) {
        printf("  >> 버스 환승 성공! 시간 손실 없음. (콤보 +1)\n");
        (*combo)++;
    }
    else {
        printf("  >> 환승 실패! 다음 버스 대기. 시간 +15분 (콤보 초기화)\n");
        *time += 15;
        *combo = 0;
    }
    wait(700);
}

/* 선택지별 기본 미니게임 무조건 실행 */
void playBasicMiniGame(int choice, int difficulty, int* hp, int* time, int* combo) {
    if (choice == 1) {
        subwayTransfer(time, difficulty, combo);
    }
    else if (choice == 2) {
        busTransfer(time, difficulty, combo);
    }
    else if (choice == 3) {
        int ok;

        printf("\n");
        printf("  ------------------------------------------\n");
        printf("  *  자전거 기본 미션: 페달 빨리 밟기!\n");
        printf("  ------------------------------------------\n");
        wait(500);

        ok = pedalGame(difficulty);

        if (ok) {
            printf("\n  >> 페달 미션 성공! 시간 -5분 (콤보 +1)\n");
            *time -= 5;
            if (*time < 0) *time = 0;
            (*combo)++;
        }
        else {
            printf("\n  >> 페달 미션 실패! 체력 -10, 시간 +5분 (콤보 초기화)\n");
            *hp -= 10;
            *time += 5;
            *combo = 0;
        }
        wait(700);
    }
    else if (choice == 4) {
        int totalObs;
        int cleared;

        printf("\n");
        printf("  ------------------------------------------\n");
        printf("  *  도보 기본 미션: 장애물 피하기!\n");
        printf("  ------------------------------------------\n");
        wait(500);

        totalObs = 4;
        cleared = jumpGame(difficulty);

        if (cleared == totalObs) {
            printf("\n  >> 장애물 미션 성공! 시간 -5분 (콤보 +1)\n");
            *time -= 5;
            if (*time < 0) *time = 0;
            (*combo)++;
        }
        else {
            printf("\n  >> 장애물 미션 실패! 체력 -15, 시간 +10분 (콤보 초기화)\n");
            *hp -= 15;
            *time += 10;
            *combo = 0;
        }
        wait(700);
    }
}

/* 기본 미니게임과 별개로, 교통수단에 맞는 추가 돌발상황이 랜덤 발생 */
void randomEvent(int* hp, int* money, int* time, int umbrella,
    int difficulty, int* combo, int choice) {
    int chance;
    int r;
    int eventType;

    (void)combo;

   
/*무조건 돌발상황 발생*/
    printf("\n");
    printf("  **********************************************\n");
    printf("  *           !! 추가 돌발상황 발생 !!          *\n");
    printf("  **********************************************\n");
    wait(700);

    eventType = rand() % 3;

    if (choice == 1) {
        if (eventType == 0) {
            printf("  *  지하철에 사람이 너무 많습니다!\n");
            printf("  >> 체력 -8, 시간 +5분\n");
            *hp -= 8;
            *time += 5;
        }
        else if (eventType == 1) {
            printf("  *  출구를 잘못 찾아서 돌아갔습니다!\n");
            printf("  >> 시간 +10분\n");
            *time += 10;
        }
        else {
            printf("  *  열차 안이 답답해서 컨디션이 떨어졌습니다!\n");
            printf("  >> 체력 -6\n");
            *hp -= 6;
        }
    }
    else if (choice == 2) {
        if (eventType == 0) {
            printf("  *  도로 정체가 심합니다!\n");
            printf("  >> 시간 +10분\n");
            *time += 10;
        }
        else if (eventType == 1) {
            printf("  *  버스가 급정거했습니다!\n");
            printf("  >> 체력 -10\n");
            *hp -= 10;
        }
        else {
            printf("  *  정류장을 한 정거장 늦게 내렸습니다!\n");
            printf("  >> 시간 +8분, 체력 -5\n");
            *time += 8;
            *hp -= 5;
        }
    }
    else if (choice == 3) {
        if (eventType == 0) {
            printf("  *  갑자기 맞바람이 강하게 붑니다!\n");
            printf("  >> 체력 -10, 시간 +5분\n");
            *hp -= 10;
            *time += 5;
        }
        else if (eventType == 1) {
            printf("  *  자전거 체인이 잠깐 빠졌습니다!\n");
            printf("  >> 시간 +10분\n");
            *time += 10;
        }
        else {
            printf("  *  자전거 도로에 사람이 많아 속도를 줄였습니다!\n");
            printf("  >> 시간 +6분\n");
            *time += 6;
        }
    }
    else if (choice == 4) {
        if (eventType == 0) {
            printf("  *  신호등에 계속 걸렸습니다!\n");
            printf("  >> 시간 +8분\n");
            *time += 8;
        }
        else if (eventType == 1) {
            if (umbrella > 0) {
                printf("  *  갑자기 비가 왔지만 우산이 있습니다!\n");
                printf("  >> 우산 덕분에 피해 없음\n");
            }
            else {
                printf("  *  갑자기 비가 쏟아졌습니다!\n");
                printf("  >> 체력 -10, 시간 +5분\n");
                *hp -= 10;
                *time += 5;
            }
        }
        else {
            printf("  *  길을 잠깐 잘못 들어 돌아갔습니다!\n");
            printf("  >> 시간 +7분, 체력 -4\n");
            *time += 7;
            *hp -= 4;
        }
    }

    printf("  **********************************************\n");
    wait(900);
}

int calculateScore(int hp, int money, int time, int maxCombo) {
    int score = 1000;

    score += hp * 3;
    score += money / 100;
    score -= time * 5;
    score += maxCombo * 150;

    if (score < 0) score = 0;

    return score;
}

void saveRank(char name[], int score) {
    Rank ranks[MAX_RANK + 1];
    int count = 0;
    FILE* fp;
    int i, j;

    fp = fopen("rank.txt", "r");

    if (fp != NULL) {
        while (count < MAX_RANK &&
            fscanf(fp, "%s %d", ranks[count].name, &ranks[count].score) == 2) {
            count++;
        }
        fclose(fp);
    }

    strcpy(ranks[count].name, name);
    ranks[count].score = score;
    count++;

    for (i = 0; i < count - 1; i++) {
        for (j = i + 1; j < count; j++) {
            if (ranks[i].score < ranks[j].score) {
                Rank temp = ranks[i];
                ranks[i] = ranks[j];
                ranks[j] = temp;
            }
        }
    }

    fp = fopen("rank.txt", "w");

    if (fp != NULL) {
        for (i = 0; i < count && i < MAX_RANK; i++) {
            fprintf(fp, "%s %d\n", ranks[i].name, ranks[i].score);
        }
        fclose(fp);
    }
}

void showRank() {
    FILE* fp;
    char name[20];
    int score;
    int rank = 1;

    printf("\n");
    printf("  +==================================================+\n");
    printf("  |                  TOP 10  랭킹                    |\n");
    printf("  +==================================================+\n");
    printf("  |  순위        이름                  점수          |\n");
    printf("  +--------------------------------------------------+\n");

    fp = fopen("rank.txt", "r");

    if (fp == NULL) {
        printf("  |          아직 랭킹이 없습니다.                   |\n");
        printf("  +==================================================+\n");
        return;
    }

    while (fscanf(fp, "%s %d", name, &score) == 2) {
        char mark[8] = "  ";

        if (rank == 1) strcpy(mark, "1st");
        else if (rank == 2) strcpy(mark, "2nd");
        else if (rank == 3) strcpy(mark, "3rd");

        printf("  |  %-4s %2d위   %-18s  %6d점     \n", mark, rank, name, score);
        rank++;
    }

    fclose(fp);
    printf("  +==================================================+\n");
}

void playGame() {
    char name[20];
    int hp, money, totalTime = 0;
    int choice, difficulty, stage, score;
    int drink = 0, coupon = 0, umbrella = 0;
    int combo = 0, maxCombo = 0;
    int useCoupon;   /* 이번 이동에 할인권 사용 여부 */

    char stages[4][60] = {
        "1단계  집  ->  버스정류장",
        "2단계  버스정류장  ->  지하철역",
        "3단계  지하철역  ->  환승역",
        "4단계  환승역  ->  학교"
    };

    clearScreen();

    printf("\n");
    printf("  ##########################################################\n");
    printf("  #              등 교  생 존  게 임  시 작                #\n");
    printf("  ##########################################################\n");

    printf("\n  이름을 입력하세요: ");
    fgets(name, sizeof(name), stdin);
    name[strcspn(name, "\n")] = '\0';

    while (1) {
        printf("\n  난이도를 선택하세요. (높을수록 미니게임이 어렵고 점수가 큼)\n");
        printf("    1. 쉬움   (체력 120 / 돈 12000원 / 타이밍 여유)\n");
        printf("    2. 보통   (체력 100 / 돈 10000원 / 타이밍 보통)\n");
        printf("    3. 어려움 (체력  80 / 돈  8000원 / 타이밍 빡빡)\n");
        printf("  선택: ");

        difficulty = inputNumber();

        if (difficulty >= 1 && difficulty <= 3) break;

        printf("  >> 잘못된 선택입니다.\n");
    }

    if (difficulty == 1) {
        hp = 120;
        money = 12000;
    }
    else if (difficulty == 2) {
        hp = 100;
        money = 10000;
    }
    else {
        hp = 80;
        money = 8000;
    }

    for (stage = 0; stage < 4; stage++) {
        while (1) {
            clearScreen();

            printf("\n");
            printf("  ==========================================================\n");
            printf("    %s\n", stages[stage]);
            printf("  ==========================================================\n\n");

            printStatus(hp, money, totalTime, combo);
            printItem(drink, coupon, umbrella);

            printf("\n  이동 수단을 선택하세요.\n");
            printf("    1. 지하철  (시간+20 / 체력-5  / 요금 1500 / 기본게임: 환승 타이밍)\n");
            printf("    2. 버스    (시간+30 / 체력-10 / 요금 1200 / 기본게임: 환승 타이밍)\n");
            printf("    3. 자전거  (시간+25 / 체력-20 / 무료 / 기본게임: 페달 연타)\n");
            printf("    4. 도보    (시간+40 / 체력-30 / 무료 / 기본게임: 장애물 피하기)\n");
            printf("    5. 에너지 드링크 사용 (체력+30)\n");
            printf("  선택: ");

            choice = inputNumber();

            if (choice >= 1 && choice <= 5) break;

            printf("  >> 잘못된 선택입니다.\n");
            wait(600);
        }

        if (choice == 5) {
            if (drink > 0) {
                printf("\n  >> 에너지 드링크 사용! 체력 +30\n");
                hp += 30;
                drink--;
            }
            else {
                printf("\n  >> 에너지 드링크가 없습니다.\n");
            }

            wait(800);
            stage--;
            continue;
        }

        /* 유료 교통수단(지하철/버스) 선택 시, 할인권이 있으면 사용 여부를 직접 물어봄 */
        useCoupon = 0;
        if ((choice == 1 || choice == 2) && coupon > 0) {
            int ans;
            printf("\n  할인권이 %d개 있습니다. 이번 이동에 사용할까요?\n", coupon);
            if (choice == 1) printf("    -> 사용 시 지하철 요금 1500원 → 700원\n");
            else             printf("    -> 사용 시 버스 요금 1200원 → 600원\n");
            printf("    1. 사용한다\n");
            printf("    2. 아낀다\n");
            printf("  선택: ");
            while (1) {
                ans = inputNumber();
                if (ans == 1 || ans == 2) break;
                printf("  >> 1 또는 2를 선택하세요: ");
            }
            if (ans == 1) useCoupon = 1;
        }

        if (choice == 1) {
            hp -= 5;
            if (useCoupon) { money -= 700; coupon--; printf("\n  >> 할인권 사용! 요금 700원\n"); }
            else { money -= 1500; }
            totalTime += 20;
        }
        else if (choice == 2) {
            hp -= 10;
            if (useCoupon) { money -= 600; coupon--; printf("\n  >> 할인권 사용! 요금 600원\n"); }
            else { money -= 1200; }
            totalTime += 30;
        }
        else if (choice == 3) {
            hp -= 20;
            totalTime += 25;
        }
        else if (choice == 4) {
            hp -= 30;
            totalTime += 40;
        }

        moveAnimation(choice);

        /* 1~4번 선택지는 선택할 때마다 기본 미니게임 무조건 실행 */
        playBasicMiniGame(choice, difficulty, &hp, &totalTime, &combo);

        /* 기본 미니게임과 별개로, 교통수단에 맞는 돌발상황 랜덤 발생 */
        randomEvent(&hp, &money, &totalTime, umbrella, difficulty, &combo, choice);

        if (combo > maxCombo) maxCombo = combo;

        randomItem(&drink, &coupon, &umbrella);

        if (hp <= 0 || money < 0) {
            clearScreen();

            printf("\n");
            printf("  XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n");
            printf("  X              등 교  실 패 !!            X\n");
            printf("  XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n");

            if (hp <= 0) {
                printf("\n  엔딩: 체력이 0이 되어 쓰러졌습니다...\n");
            }
            else {
                printf("\n  엔딩: 돈이 부족해 더 갈 수 없습니다...\n");
            }

            score = calculateScore(hp, money, totalTime, maxCombo);

            printf("\n  최종 점수: %d점 (최고 콤보 %d)\n", score, maxCombo);

            saveRank(name, score);
            showRank();

            printf("\n  (엔터를 누르면 메뉴로 돌아갑니다) ");
            getchar();

            return;
        }

        wait(600);
    }

    clearScreen();

    printf("\n");
    printf("  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$\n");
    printf("  $          학 교  도 착  성 공 !!          $\n");
    printf("  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$\n\n");

    printStatus(hp, money, totalTime, combo);

    score = calculateScore(hp, money, totalTime, maxCombo);

    printf("\n  최종 점수: %d점 (최고 콤보 %d)\n", score, maxCombo);

    if (totalTime <= 90 && hp >= 60 && maxCombo >= 3) {
        printf("  엔딩:  [ S급 ] 실력으로 완벽한 등교!\n");
    }
    else if (totalTime <= 120) {
        printf("  엔딩:  [ A급 ] 무난한 등교!\n");
    }
    else if (hp < 30) {
        printf("  엔딩:  [ C급 ] 지친 상태로 간신히 도착...\n");
    }
    else {
        printf("  엔딩:  [ B급 ] 늦었지만 도착!\n");
    }

    saveRank(name, score);
    showRank();

    printf("\n  (엔터를 누르면 메뉴로 돌아갑니다) ");
    getchar();
}

int main(void) {
    int menu;

    srand((unsigned int)time(NULL));

    while (1) {
        clearScreen();
        printTitle();

        printf("    1. 게임 시작\n");
        printf("    2. 랭킹 보기\n");
        printf("    0. 게임 종료\n");
        printf("\n  선택: ");

        menu = inputNumber();

        if (menu == 1) {
            playGame();
        }
        else if (menu == 2) {
            showRank();
            printf("\n  (엔터를 누르면 메뉴로 돌아갑니다) ");
            getchar();
        }
        else if (menu == 0) {
            remove("rank.txt");
            printf("\n  랭킹 기록을 초기화합니다.\n");
            printf("  게임을 종료합니다. 안녕히 가세요!\n");
            break;
        }
        else {
            printf("\n  >> 잘못된 선택입니다.\n");
            wait(800);
        }
    }

    return 0;
}
