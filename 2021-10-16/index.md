# 洛谷模板题目练习


为了学习C++顺便练手，听67建议先刷一遍洛谷的模板题来练手，在这里记录写题过程和总结。

## 题目

1. P3367 【模板】并查集(2021/10/16)：

   本题目理解起来不太难。主要是要看属于哪个集合，即找一个节点的根节点，在题解中有人提到了**路径压缩**，这个知识点我已经记不得了，在这里回忆一下[并查集路径压缩 | 菜鸟教程 (runoob.com)](https://www.runoob.com/data-structures/union-find-compress.html)。下面是AC的代码：

   ```c++
   #include <bits/stdc++.h>
   using namespace std;
   
   int n, m, z, x, y, f[10001];
   
   // 找父节点，也就是属于哪个集合 
   int find(int x){
   	if(f[x] == x){
   		return x; // 元素的父节点是自己，则返回本身 
   	}
   	else{
   		f[x] = find(f[x]);//直到找到父节点为止 
   	} 
   	return f[x];
   }
   
   void merge(int x, int y){
   	f[find(y)] = find(x); // 父节点一致 
   }
   
   void search(int x, int y){
   	if(find(x) == find(y)){
   		cout<<"Y"<<endl;
   	}else{
   		cout<<"N"<<endl;
   	}
   } 
    
   int main(){
   	cin>>n>>m;
   	int i;
   	for(i = 1; i <=n ; i++){
   		f[i] = i; //初始化并查集 
   	}
   	while(m--){
   		cin>>z>>x>>y;
   		if(z == 1){
   			merge(x,y);
   		} 
   		if(z == 2){
   			search(x,y);
   		}
   	} 
   	return 0;
   }
   ```

   

2. P3366 【模板】最小生成树：

   忘记怎么求最小生成树了，随便查了一篇博客[图的最小生成树 - 智者侬哥 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wuxiangnong/p/10885129.html)学习，有kruskal和prim两种算法，两者好像都是运用贪心算法，kruskal是直接将边排序，然后选能够组成生成树的边，prim是不断找最小值添加新节点。前者好像适合稀疏图，后者适合稠密图，[最小生成树算法 - Nemlit 的博客 - 洛谷博客 (luogu.com.cn)](https://www.luogu.com.cn/blog/tbr-blog/solution-p3366)。正好在kruskal方法里判断是否需要添加某条边时也用到了并查集。

3. 

