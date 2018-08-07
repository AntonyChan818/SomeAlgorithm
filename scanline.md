# Scanline Filling  
- Introduction:  
扫描线填充算法的基本思想是：用水平扫描线从上到下（或从下到上）扫描由多条首尾相连的线段构成的多边形，每根扫描线与多边形的某些边产生一系列交点。将这些交点按照x坐标排序，将排序后的点两两成对，作为线段的两个端点，以所填的颜色画水平直线。多边形被扫描完毕后，颜色填充也就完成了。扫描线填充算法也可以归纳为以下4个步骤：  
（1）求交点。即扫描线和多边形的交点。  
（2）交点排序。  
（3）对排序后的点两两匹配。  
（4）更新扫描线，判断是否完成多边形扫描。

- 求交点  
（1） 每次只有相关的几条边可能与扫描线有交点，不必对所有的边进行求交计算；  
（2）相邻的扫描线与同一直线段的交点存在步进关系。  
第一个特点是显而易见的，为了减少计算量，扫描线算法需要维护一张“活动边Active Edge组成的表，称为“活化边表Active Edge Table（AET）”。例如扫描线4的“活化边表”  p1p2和p3p4两条边组成。  
第二个特点，可以进步证明：假设当前扫描线与多边形的某一条边的交点已经通过直线段求交算法计算出来，得到交点的坐标为（x, y），则下一条扫描线与这条边的交点不需要再求交计算，通过步进关系可以直接得到新交点坐标为（x + △x, y + 1）。△x为y增加1个单位时，x增加的单位。即多边形边界的斜率的倒数。  
“活动边表”是扫描线填充算法的核心，保证了填充时不需太多的求交点运算。为了方便活性边表的建立与更新，还需要建立一个边表NET（New Edge Table）来存放所有的边。  
扫描线填充算法只关注交点的x坐标，即交点的横坐标；处理下一条扫描线交点时，根据 △x直接得出；还需要边的最大y坐标y_max作为活边表和扫描结束的判断依据；了便于插入和修改，定义为链表结构。  
```
typedef struct tagEdge{  
    int y_max;     // 边的最大y坐标  
    float x;       // 与扫描线交点x坐标  
    float dx;      // 斜率的倒数，Δx  
    Edge * pNext;  // 下一条边  
} Edge;  

//扫描线填充算法  
CPtrArray & CShapeFiller::scanLineFill(CPtrArray &m_tranArrShape){  
    CPtrArray *pArr = new CPtrArray();  
      
    float y_max = (float)INT_MIN;    // y坐标值的最大值  
    float y_min = (float)INT_MAX;    // y坐标值的最小值  
  
    //获取y坐标值最大和最小值  
    GetYMaxMin(y_max, y_min);  
  
    //最大最小值取整  
    int iy_max = (int)y_max;  
    int iy_min = (int)y_min;  
  
    //y_max 取整的误差  
    float y_deviation = y_min - (float)iy_min;  
  
/* -------------------------- 定义边表NET及活化边表AET ---------------------- */  
    //活化边表AET:元素与当前扫描线相交  基本元素：边Edge.   
    //边表NET: 按边的下端点的Y坐标对非水平的边指针数组  
    Edge *pAET=NULL;    // 活化边表的表头指针  
    Edge **pNET=NULL;   // 边表的表头指针  
  
    pAET = new Edge();  //初始化活动表头指针，第一个元素不用  
    pAET->pNext = NULL;  
  
/* ---------------------- 初始化边表NET --------------------------- */  
  
    //边表NET数组的长度   
    int length = (iy_max-iy_min);   
  
    // 初始化边表NET，第一个元素不用  
    pNET=new Edge*[length + 1];  
  
    //初始化边表NET中的每一个指针元素，赋予内存空间  
    for(int i = 0;i <= length;i++)  
    {  
        pNET[i] = new Edge();  
        pNET[i]->pNext = NULL;  
    }  
  
    //将所有的边添加到NET边表中  
    CreateNET(pNET,iy_min);  
     
/* --------------------------- 扫描线填充 ----------------------- */  
    // 扫描线填充，从最小y坐标开始扫描，下闭上开  
    for(int i=iy_min;i < iy_max;i += 1)  
    {  
        //活动边表AET的头指针  
        Edge *pEdgeFirst=pNET[i-iy_min];  
  
        //初始化活化边表AET  
        InitializeAET(pEdgeFirst,pAET);  
  
        //对当前AET活化边表进行X坐标值升序排序  
        SortAcendX(pAET);  
  
        //添加间距参数判断  
        //若间距为0 ，不填充  
        if (lineSpacing ==0)  
        {  
            for(int i=0;i <=length;i++)  
                if(pNET[i])  
                    delete pNET[i];  
  
            if(pAET)   
                delete pAET;  
            if(pNET)  
                delete[] pNET;   
  
            //释放指针申请的内存空间  
            m_tranArrShape.RemoveAll();  
            return pArr;  
        }  
        else  
        {    
            //间距不为0，填充实现  
            if(i%lineSpacing==0)  
            {  
                  
                // 遍历活边表，将坐标点存入CPtrArray对象中  
                SavePoints(pAET,i,*pArr,y_deviation,x0,y0);  
            }  
  
        }  
  
        // 更新扫描线  
        UpdateScanLine(pAET,i);  
  
    }  
  
    // 删除边表  
    for(int i=0;i <=length;i++)  
        if(pNET[i])  
            delete pNET[i];  
  
    if(pAET)   
        delete pAET;  
    if(pNET)  
        delete[] pNET;   
  
    //释放指针申请的内存空间  
    m_tranArrShape.RemoveAll();  
  
    return *pArr;  
}  
```  

