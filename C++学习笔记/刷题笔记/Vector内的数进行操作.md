# Vector内的数进行操作

```c++
vector<NestedInteger> nestedList
```

假设需要判断里面是否是整数

```c++
nestedList.isInteger();
```

判断取得其中的Vector：

```c++
nestedList.getList();
```

深度优先对vector进行求取：

```c++
vector<int> res;
        vector<int>::iterator cur;
       void dfs(vector<NestedInteger> &nestedList){
           for(auto &nest: nestedList){
               if(nest.isInteger()){
                   res.push_back(nest.getInteger());
               }else{
                   dfs(nest.getList());
               }
           }
       }
```

