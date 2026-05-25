#include <stdio.h>
#include <stdlib.h>

#define MAX_VERTEX_NUM 20
#define INFINITY 65535

// 图的类型定义
typedef enum {DG, DN, UDG, UDN} GraphKind;  // 有向图，有向网，无向图，无向网
typedef char VertexType;   // 顶点数据类型
typedef int EdgeType;      // 边权值类型

// 图的结构定义（邻接矩阵表示法）
typedef struct {
    VertexType vexs[MAX_VERTEX_NUM];           // 顶点数组
    EdgeType arcs[MAX_VERTEX_NUM][MAX_VERTEX_NUM];  // 邻接矩阵
    int vexNum, edgeNum;                       // 当前顶点数和边数
    GraphKind kind;                           // 图的种类
    int isCreated;                           // 标记图是否已创建
} MGraph;

// 队列结构（用于BFS遍历）
typedef struct {
    int data[MAX_VERTEX_NUM];
    int front, rear;
} Queue;

// 全局变量声明
MGraph graph;
int visited[MAX_VERTEX_NUM];  // DFS访问标记数组

// 函数声明
void initGraph();
int locateVex(VertexType v);
void createGraph();
void printGraph();
void printMenu();
void dfsTraverse();
void dfs(int v);
void bfsTraverse();
void initQueue(Queue *Q);
int queueEmpty(Queue Q);
void enQueue(Queue *Q, int e);
void deQueue(Queue *Q, int *e);
void findAdjacentVex();
void getVexValue();
void setVexValue();
void insertVex();
void deleteVex();
void insertEdge();
void deleteEdge();

int main() {
    int choice;
    char continueFlag = 'Y';
    
    // 初始化图
    initGraph();
    
    printf("==============================================\n");
    printf("          图的可交互操作程序\n");
    printf("    实现邻接矩阵表示及遍历算法\n");
    printf("==============================================\n");
    
    do {
        printMenu();
        printf("请选择操作(0-12): ");
        scanf("%d", &choice);
        getchar();  // 清除输入缓冲区
        
        switch(choice) {
            case 0:
                printf("感谢使用，再见！\n");
                return 0;
                
            case 1:
                createGraph();
                break;
                
            case 2:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    printGraph();
                }
                break;
                
            case 3:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    dfsTraverse();
                }
                break;
                
            case 4:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    bfsTraverse();
                }
                break;
                
            case 5:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    findAdjacentVex();
                }
                break;
                
            case 6:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    getVexValue();
                }
                break;
                
            case 7:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    setVexValue();
                }
                break;
                
            case 8:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    insertVex();
                }
                break;
                
            case 9:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    deleteVex();
                }
                break;
                
            case 10:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    insertEdge();
                }
                break;
                
            case 11:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    deleteEdge();
                }
                break;
                
            case 12:
                if (!graph.isCreated) {
                    printf("请先创建图！\n");
                } else {
                    printf("当前图信息：\n");
                    printf("顶点数: %d\n", graph.vexNum);
                    printf("边数: %d\n", graph.edgeNum);
                    printf("图类型: ");
                    switch(graph.kind) {
                        case DG: printf("有向图\n"); break;
                        case DN: printf("有向网\n"); break;
                        case UDG: printf("无向图\n"); break;
                        case UDN: printf("无向网\n"); break;
                    }
                }
                break;
                
            default:
                printf("无效的选择，请重新输入！\n");
        }
        
        printf("\n是否继续操作？(Y/N): ");
        scanf("%c", &continueFlag);
        getchar();  // 清除换行符
        
    } while(continueFlag == 'Y' || continueFlag == 'y');
    
    printf("感谢使用，再见！\n");
    return 0;
}

// 初始化图
void initGraph() {
    graph.isCreated = 0;
    graph.vexNum = 0;
    graph.edgeNum = 0;
}

// 打印菜单
void printMenu() {
    printf("\n==============================================\n");
    printf("                主菜单\n");
    printf("==============================================\n");
    printf("1. 创建图\n");
    printf("2. 打印邻接矩阵\n");
    printf("3. 深度优先遍历(DFS)\n");
    printf("4. 广度优先遍历(BFS)\n");
    printf("5. 查找顶点的邻接点\n");
    printf("6. 获取顶点值\n");
    printf("7. 设置顶点值\n");
    printf("8. 插入顶点\n");
    printf("9. 删除顶点\n");
    printf("10. 插入边\n");
    printf("11. 删除边\n");
    printf("12. 显示图信息\n");
    printf("0. 退出程序\n");
    printf("----------------------------------------------\n");
}

// 查找顶点在顶点数组中的位置
int locateVex(VertexType v) {
    for (int i = 0; i < graph.vexNum; i++) {
        if (graph.vexs[i] == v) {
            return i;
        }
    }
    return -1;  // 未找到
}

// 创建图
void createGraph() {
    int i, j, k;
    VertexType v1, v2;
    EdgeType weight;
    
    if (graph.isCreated) {
        printf("图已存在，是否重新创建？(Y/N): ");
        char confirm;
        scanf("%c", &confirm);
        getchar();
        if (confirm != 'Y' && confirm != 'y') {
            return;
        }
    }
    
    printf("\n========== 创建图 ==========\n");
    
    // 选择图类型
    printf("请选择图的类型:\n");
    printf("0. 有向图(DG)\n");
    printf("1. 有向网(DN)\n");
    printf("2. 无向图(UDG)\n");
    printf("3. 无向网(UDN)\n");
    printf("请选择(0-3): ");
    int type;
    scanf("%d", &type);
    getchar();
    
    graph.kind = (GraphKind)type;
    
    // 输入顶点数
    do {
        printf("请输入顶点数(1-%d): ", MAX_VERTEX_NUM);
        scanf("%d", &graph.vexNum);
        getchar();
        if (graph.vexNum < 1 || graph.vexNum > MAX_VERTEX_NUM) {
            printf("顶点数必须在1-%d之间！\n", MAX_VERTEX_NUM);
        }
    } while(graph.vexNum < 1 || graph.vexNum > MAX_VERTEX_NUM);
    
    // 输入顶点信息
    printf("\n请输入%d个顶点的值(建议用大写字母A,B,C...):\n", graph.vexNum);
    for (i = 0; i < graph.vexNum; i++) {
        printf("顶点%d: ", i+1);
        scanf("%c", &graph.vexs[i]);
        getchar();
        
        // 检查顶点是否重复
        for (j = 0; j < i; j++) {
            if (graph.vexs[j] == graph.vexs[i]) {
                printf("顶点%c已存在，请重新输入: ", graph.vexs[i]);
                i--;
                break;
            }
        }
    }
    
    // 初始化邻接矩阵
    for (i = 0; i < graph.vexNum; i++) {
        for (j = 0; j < graph.vexNum; j++) {
            if (graph.kind == DG || graph.kind == UDG) {
                graph.arcs[i][j] = 0;  // 图
            } else {
                graph.arcs[i][j] = INFINITY;  // 网
            }
        }
    }
    
    // 输入边数
    printf("\n请输入边数(0-%d): ", graph.vexNum*(graph.vexNum-1)/2);
    scanf("%d", &graph.edgeNum);
    getchar();
    
    if (graph.edgeNum > 0) {
        printf("\n请输入%d条边:\n", graph.edgeNum);
        if (graph.kind == DN || graph.kind == UDN) {
            printf("格式: 起点 终点 权值(例如: A B 5)\n");
        } else {
            printf("格式: 起点 终点(例如: A B)\n");
        }
    }
    
    // 输入边信息
    for (k = 0; k < graph.edgeNum; k++) {
        int index1, index2;
        
        if (graph.kind == DN || graph.kind == UDN) {
            printf("边%d: ", k+1);
            scanf("%c %c %d", &v1, &v2, &weight);
            getchar();
        } else {
            printf("边%d: ", k+1);
            scanf("%c %c", &v1, &v2);
            getchar();
            weight = 1;  // 图的边权值为1
        }
        
        index1 = locateVex(v1);
        index2 = locateVex(v2);
        
        if (index1 == -1) {
            printf("顶点%c不存在，请重新输入！\n", v1);
            k--;
            continue;
        }
        if (index2 == -1) {
            printf("顶点%c不存在，请重新输入！\n", v2);
            k--;
            continue;
        }
        if (index1 == index2) {
            printf("起点和终点不能相同，请重新输入！\n");
            k--;
            continue;
        }
        
        // 设置边
        switch (graph.kind) {
            case DG:  // 有向图
                graph.arcs[index1][index2] = 1;
                break;
            case DN:  // 有向网
                graph.arcs[index1][index2] = weight;
                break;
            case UDG:  // 无向图
                graph.arcs[index1][index2] = 1;
                graph.arcs[index2][index1] = 1;
                break;
            case UDN:  // 无向网
                graph.arcs[index1][index2] = weight;
                graph.arcs[index2][index1] = weight;
                break;
        }
    }
    
    graph.isCreated = 1;
    printf("\n图创建成功！\n");
    printf("顶点数: %d, 边数: %d\n", graph.vexNum, graph.edgeNum);
}

// 打印邻接矩阵
void printGraph() {
    int i, j;
    
    printf("\n========== 图的邻接矩阵 ==========\n");
    printf("图类型: ");
    switch(graph.kind) {
        case DG: printf("有向图\n"); break;
        case DN: printf("有向网\n"); break;
        case UDG: printf("无向图\n"); break;
        case UDN: printf("无向网\n"); break;
    }
    printf("顶点数: %d, 边数: %d\n", graph.vexNum, graph.edgeNum);
    
    // 打印顶点
    printf("顶点: ");
    for (i = 0; i < graph.vexNum; i++) {
        printf("%c ", graph.vexs[i]);
    }
    printf("\n\n");
    
    // 打印表头
    printf("     ");
    for (i = 0; i < graph.vexNum; i++) {
        printf("%3c ", graph.vexs[i]);
    }
    printf("\n");
    
    // 打印分隔线
    printf("    +");
    for (i = 0; i < graph.vexNum; i++) {
        printf("----");
    }
    printf("\n");
    
    // 打印邻接矩阵
    for (i = 0; i < graph.vexNum; i++) {
        printf("  %c| ", graph.vexs[i]);
        for (j = 0; j < graph.vexNum; j++) {
            if (graph.kind == DG || graph.kind == UDG) {
                printf("%3d ", graph.arcs[i][j]);
            } else {
                if (graph.arcs[i][j] == INFINITY) {
                    printf(" INF ");
                } else {
                    printf("%3d ", graph.arcs[i][j]);
                }
            }
        }
        printf("\n");
    }
    
    // 打印图的结构说明
    printf("\n说明:\n");
    printf("1. 0/INF表示无边，1/数字表示有边/权值\n");
    printf("2. 有向图/网: 行表示起点，列表示终点\n");
    printf("3. 无向图/网: 矩阵对称\n");
}

// 深度优先遍历辅助函数
void dfs(int v) {
    int w;
    printf("%c ", graph.vexs[v]);
    visited[v] = 1;
    
    for (w = 0; w < graph.vexNum; w++) {
        if ((graph.kind == UDG || graph.kind == UDN) && 
            graph.arcs[v][w] != 0 && graph.arcs[v][w] != INFINITY) {
            if (!visited[w]) {
                dfs(w);
            }
        } else if ((graph.kind == DG || graph.kind == DN) && 
                   graph.arcs[v][w] != 0 && graph.arcs[v][w] != INFINITY) {
            if (!visited[w]) {
                dfs(w);
            }
        }
    }
}

// 深度优先遍历
void dfsTraverse() {
    VertexType startVex;
    int startIndex;
    
    // 初始化访问标记数组
    for (int i = 0; i < graph.vexNum; i++) {
        visited[i] = 0;
    }
    
    printf("\n========== 深度优先遍历(DFS) ==========\n");
    printf("请输入起始顶点: ");
    scanf("%c", &startVex);
    getchar();
    
    startIndex = locateVex(startVex);
    if (startIndex == -1) {
        printf("顶点%c不存在！\n", startVex);
        return;
    }
    
    printf("从顶点%c开始的DFS遍历序列: ", startVex);
    dfs(startIndex);
    
    // 检查非连通图
    int hasUnvisited = 0;
    for (int i = 0; i < graph.vexNum; i++) {
        if (!visited[i]) {
            if (!hasUnvisited) {
                printf("\n图是非连通的，继续遍历其余顶点: ");
                hasUnvisited = 1;
            }
            dfs(i);
        }
    }
    printf("\n");
}

// 队列操作函数
void initQueue(Queue *Q) {
    Q->front = Q->rear = 0;
}

int queueEmpty(Queue Q) {
    return Q.front == Q.rear;
}

void enQueue(Queue *Q, int e) {
    if ((Q->rear + 1) % MAX_VERTEX_NUM == Q->front) {
        printf("队列已满！\n");
        return;
    }
    Q->data[Q->rear] = e;
    Q->rear = (Q->rear + 1) % MAX_VERTEX_NUM;
}

void deQueue(Queue *Q, int *e) {
    if (queueEmpty(*Q)) {
        printf("队列为空！\n");
        return;
    }
    *e = Q->data[Q->front];
    Q->front = (Q->front + 1) % MAX_VERTEX_NUM;
}

// 广度优先遍历
void bfsTraverse() {
    VertexType startVex;
    int startIndex, currentVex, w;
    Queue Q;
    
    // 初始化访问标记数组
    for (int i = 0; i < graph.vexNum; i++) {
        visited[i] = 0;
    }
    
    printf("\n========== 广度优先遍历(BFS) ==========\n");
    printf("请输入起始顶点: ");
    scanf("%c", &startVex);
    getchar();
    
    startIndex = locateVex(startVex);
    if (startIndex == -1) {
        printf("顶点%c不存在！\n", startVex);
        return;
    }
    
    printf("从顶点%c开始的BFS遍历序列: ", startVex);
    
    // 初始化队列
    initQueue(&Q);
    
    // 访问起始顶点并入队
    printf("%c ", graph.vexs[startIndex]);
    visited[startIndex] = 1;
    enQueue(&Q, startIndex);
    
    while (!queueEmpty(Q)) {
        deQueue(&Q, &currentVex);
        
        // 遍历当前顶点的所有邻接点
        for (w = 0; w < graph.vexNum; w++) {
            if ((graph.kind == UDG || graph.kind == UDN) && 
                graph.arcs[currentVex][w] != 0 && graph.arcs[currentVex][w] != INFINITY) {
                if (!visited[w]) {
                    printf("%c ", graph.vexs[w]);
                    visited[w] = 1;
                    enQueue(&Q, w);
                }
            } else if ((graph.kind == DG || graph.kind == DN) && 
                       graph.arcs[currentVex][w] != 0 && graph.arcs[currentVex][w] != INFINITY) {
                if (!visited[w]) {
                    printf("%c ", graph.vexs[w]);
                    visited[w] = 1;
                    enQueue(&Q, w);
                }
            }
        }
    }
    
    // 检查非连通图
    int hasUnvisited = 0;
    for (int i = 0; i < graph.vexNum; i++) {
        if (!visited[i]) {
            if (!hasUnvisited) {
                printf("\n图是非连通的，继续遍历其余顶点: ");
                hasUnvisited = 1;
            }
            printf("%c ", graph.vexs[i]);
            visited[i] = 1;
            enQueue(&Q, i);
            
            while (!queueEmpty(Q)) {
                deQueue(&Q, &currentVex);
                for (w = 0; w < graph.vexNum; w++) {
                    if ((graph.kind == UDG || graph.kind == UDN) && 
                        graph.arcs[currentVex][w] != 0 && graph.arcs[currentVex][w] != INFINITY) {
                        if (!visited[w]) {
                            printf("%c ", graph.vexs[w]);
                            visited[w] = 1;
                            enQueue(&Q, w);
                        }
                    } else if ((graph.kind == DG || graph.kind == DN) && 
                               graph.arcs[currentVex][w] != 0 && graph.arcs[currentVex][w] != INFINITY) {
                        if (!visited[w]) {
                            printf("%c ", graph.vexs[w]);
                            visited[w] = 1;
                            enQueue(&Q, w);
                        }
                    }
                }
            }
        }
    }
    printf("\n");
}

// 查找顶点的邻接点
void findAdjacentVex() {
    VertexType v;
    int index, i, count = 0;
    
    printf("\n========== 查找顶点的邻接点 ==========\n");
    printf("请输入要查找的顶点: ");
    scanf("%c", &v);
    getchar();
    
    index = locateVex(v);
    if (index == -1) {
        printf("顶点%c不存在！\n", v);
        return;
    }
    
    printf("顶点%c的邻接点: ", v);
    for (i = 0; i < graph.vexNum; i++) {
        if ((graph.kind == UDG || graph.kind == UDN) && 
            graph.arcs[index][i] != 0 && graph.arcs[index][i] != INFINITY) {
            printf("%c ", graph.vexs[i]);
            count++;
        } else if ((graph.kind == DG || graph.kind == DN) && 
                   graph.arcs[index][i] != 0 && graph.arcs[index][i] != INFINITY) {
            printf("%c(出边) ", graph.vexs[i]);
            count++;
        }
    }
    
    if (graph.kind == DG || graph.kind == DN) {
        // 对于有向图/网，还要查找入边邻接点
        printf("\n顶点%c的入边邻接点: ", v);
        for (i = 0; i < graph.vexNum; i++) {
            if (graph.arcs[i][index] != 0 && graph.arcs[i][index] != INFINITY) {
                printf("%c ", graph.vexs[i]);
            }
        }
    }
    
    if (count == 0) {
        printf("无邻接点");
    }
    printf("\n");
}

// 获取顶点值
void getVexValue() {
    int index;
    
    printf("\n========== 获取顶点值 ==========\n");
    printf("请输入顶点序号(1-%d): ", graph.vexNum);
    scanf("%d", &index);
    getchar();
    
    if (index < 1 || index > graph.vexNum) {
        printf("顶点序号无效！\n");
        return;
    }
    
    printf("顶点%d的值为: %c\n", index, graph.vexs[index-1]);
}

// 设置顶点值
void setVexValue() {
    int index;
    VertexType newValue;
    
    printf("\n========== 设置顶点值 ==========\n");
    printf("请输入要修改的顶点序号(1-%d): ", graph.vexNum);
    scanf("%d", &index);
    getchar();
    
    if (index < 1 || index > graph.vexNum) {
        printf("顶点序号无效！\n");
        return;
    }
    
    printf("请输入新的顶点值: ");
    scanf("%c", &newValue);
    getchar();
    
    // 检查新值是否已存在
    for (int i = 0; i < graph.vexNum; i++) {
        if (i != index-1 && graph.vexs[i] == newValue) {
            printf("顶点%c已存在，不能重复！\n", newValue);
            return;
        }
    }
    
    printf("将顶点%d的值从%c修改为%c\n", index, graph.vexs[index-1], newValue);
    graph.vexs[index-1] = newValue;
    printf("修改成功！\n");
}

// 插入顶点
void insertVex() {
    VertexType newVex;
    
    if (graph.vexNum >= MAX_VERTEX_NUM) {
        printf("顶点数已达到最大值%d，无法插入新顶点！\n", MAX_VERTEX_NUM);
        return;
    }
    
    printf("\n========== 插入顶点 ==========\n");
    printf("请输入新顶点的值: ");
    scanf("%c", &newVex);
    getchar();
    
    // 检查顶点是否已存在
    if (locateVex(newVex) != -1) {
        printf("顶点%c已存在！\n", newVex);
        return;
    }
    
    // 在顶点数组末尾添加新顶点
    graph.vexNum++;
    graph.vexs[graph.vexNum-1] = newVex;
    
    // 初始化新顶点的邻接关系
    for (int i = 0; i < graph.vexNum; i++) {
        if (graph.kind == DG || graph.kind == UDG) {
            graph.arcs[graph.vexNum-1][i] = 0;
            graph.arcs[i][graph.vexNum-1] = 0;
        } else {
            graph.arcs[graph.vexNum-1][i] = INFINITY;
            graph.arcs[i][graph.vexNum-1] = INFINITY;
        }
    }
    
    printf("顶点%c插入成功！\n", newVex);
}

// 删除顶点
void deleteVex() {
    VertexType v;
    int index, i, j, k;
    
    printf("\n========== 删除顶点 ==========\n");
    printf("请输入要删除的顶点: ");
    scanf("%c", &v);
    getchar();
    
    index = locateVex(v);
    if (index == -1) {
        printf("顶点%c不存在！\n", v);
        return;
    }
    
    if (graph.vexNum <= 1) {
        printf("至少需要保留一个顶点！\n");
        return;
    }
    
    // 统计要删除的边数
    int edgeCount = 0;
    for (i = 0; i < graph.vexNum; i++) {
        if (graph.arcs[index][i] != 0 && graph.arcs[index][i] != INFINITY) {
            edgeCount++;
        }
        if (i != index && graph.arcs[i][index] != 0 && graph.arcs[i][index] != INFINITY) {
            edgeCount++;
        }
    }
    // 无向图/网每条边被统计两次
    if (graph.kind == UDG || graph.kind == UDN) {
        edgeCount /= 2;
    }
    
    // 删除顶点
    for (i = index; i < graph.vexNum - 1; i++) {
        graph.vexs[i] = graph.vexs[i+1];
    }
    
    // 删除邻接矩阵中的行和列
    for (i = index; i < graph.vexNum - 1; i++) {
        for (j = 0; j < graph.vexNum; j++) {
            graph.arcs[i][j] = graph.arcs[i+1][j];
        }
    }
    
    for (j = index; j < graph.vexNum - 1; j++) {
        for (i = 0; i < graph.vexNum - 1; i++) {
            graph.arcs[i][j] = graph.arcs[i][j+1];
        }
    }
    
    graph.vexNum--;
    graph.edgeNum -= edgeCount;
    
    printf("顶点%c删除成功！删除了%d条边。\n", v, edgeCount);
}

// 插入边
void insertEdge() {
    VertexType v1, v2;
    int index1, index2;
    EdgeType weight = 1;
    
    printf("\n========== 插入边 ==========\n");
    
    if (graph.kind == DN || graph.kind == UDN) {
        printf("格式: 起点 终点 权值\n");
        printf("请输入: ");
        scanf("%c %c %d", &v1, &v2, &weight);
        getchar();
    } else {
        printf("格式: 起点 终点\n");
        printf("请输入: ");
        scanf("%c %c", &v1, &v2);
        getchar();
    }
    
    index1 = locateVex(v1);
    index2 = locateVex(v2);
    
    if (index1 == -1) {
        printf("顶点%c不存在！\n", v1);
        return;
    }
    if (index2 == -1) {
        printf("顶点%c不存在！\n", v2);
        return;
    }
    if (index1 == index2) {
        printf("起点和终点不能相同！\n");
        return;
    }
    
    // 检查边是否已存在
    if (graph.arcs[index1][index2] != 0 && graph.arcs[index1][index2] != INFINITY) {
        printf("边(%c,%c)已存在！\n", v1, v2);
        return;
    }
    
    // 设置边
    switch (graph.kind) {
        case DG:  // 有向图
            graph.arcs[index1][index2] = 1;
            break;
        case DN:  // 有向网
            graph.arcs[index1][index2] = weight;
            break;
        case UDG:  // 无向图
            graph.arcs[index1][index2] = 1;
            graph.arcs[index2][index1] = 1;
            break;
        case UDN:  // 无向网
            graph.arcs[index1][index2] = weight;
            graph.arcs[index2][index1] = weight;
            break;
    }
    
    graph.edgeNum++;
    printf("边(%c,%c)插入成功！\n", v1, v2);
}

// 删除边
void deleteEdge() {
    VertexType v1, v2;
    int index1, index2;
    
    printf("\n========== 删除边 ==========\n");
    printf("格式: 起点 终点\n");
    printf("请输入: ");
    scanf("%c %c", &v1, &v2);
    getchar();
    
    index1 = locateVex(v1);
    index2 = locateVex(v2);
    
    if (index1 == -1) {
        printf("顶点%c不存在！\n", v1);
        return;
    }
    if (index2 == -1) {
        printf("顶点%c不存在！\n", v2);
        return;
    }
    if (index1 == index2) {
        printf("起点和终点不能相同！\n");
        return;
    }
    
    // 检查边是否存在
    if (graph.arcs[index1][index2] == 0 || graph.arcs[index1][index2] == INFINITY) {
        printf("边(%c,%c)不存在！\n", v1, v2);
        return;
    }
    
    // 删除边
    switch (graph.kind) {
        case DG:  // 有向图
            graph.arcs[index1][index2] = 0;
            break;
        case DN:  // 有向网
            graph.arcs[index1][index2] = INFINITY;
            break;
        case UDG:  // 无向图
            graph.arcs[index1][index2] = 0;
            graph.arcs[index2][index1] = 0;
            break;
        case UDN:  // 无向网
            graph.arcs[index1][index2] = INFINITY;
            graph.arcs[index2][index1] = INFINITY;
            break;
    }
    
    graph.edgeNum--;
    printf("边(%c,%c)删除成功！\n", v1, v2);
}
