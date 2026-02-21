1. 介绍 
	Ranges 库是 C++20 对传统 STL 算法（比如`sort`、`find`、`for_each`）和容器的升级—— 核心解决传统 STL 的两大痛点：
	1. 传统算法必须手动传`begin()`/`end()`迭代器，代码繁琐且易出错；
	2. 传统算法的 “中间操作”（比如过滤→转换→排序）需要创建临时容器，效率低、代码冗余。
2. 核心概念 
	* **Range（范围）**：
		* 任何能获取 “起始 / 结束迭代器” 的对象（容器、视图、子范围）
		* 例：`vector`、`string`、`views::iota(1,5)`
	* **View（视图）**
		* 对 Range 的 “虚拟包装”，不拷贝数据，只记录操作逻辑（轻量级、延迟计算）
		* 例：`views::filter`（过滤）、`views::transform`（转换）
	* **Adaptor（适配器）**
		* 用于生成 View 的 “操作符”，通过`|`（管道符）串联，实现 “过滤→转换→切片” 等操作
		* 例：`v|views::filter(...)|views::transform(...)`
3. 用法
	1. 头文件
		```cpp
		#include <ranges> //核心：Range、View、Adaptor（必包含）
		#include <algorithm> //Ranges版算法（比如ranges::sort、ranges::find）
		```
	2. 对象 
		任何满足 “能调用`begin()`/`end()`” 的对象
		- 标准容器：`vector`、`list`、`set`、`string`、`array`
		- 原生数组：`int arr[5] = {1,2,3,4,5}`
		- 视图：`views::iota(1, 10)`（生成 1~9 的序列）
	3. View 适配器
		* `views::filter`
			* 过滤元素（保留满足条件的）
			* 例：`v|views::filter ([](int x) { return x%2==0; })` // 保留偶数
		* `views::transform`
			* 转换元素（修改 / 映射元素）
			* 例：`v|views::transform ([](int x) { return x*2; })` // 元素翻倍
		* `views::take`
			* 取前 n 个元素
			* 例： `v|views::take (3)` // 取前 3 个元素
		* `views::drop`
			* 跳过前 n 个元素
			* 例：`v|views::drop (2)` // 跳过前 2 个元素
		* `views::reverse`
			* 反转元素顺序
			* 例：`v|views::reverse` // 反转 
		* `views::iota`
			* 生成连续整数序列（延迟生成）
			* 例：`views::iota(1, 6)` // 生成 1,2,3,4,5
		* `views::slice`
			* 切片（取 [start, end) 区间元素）
			* 例：`v|views::slice (1,4)` // 取索引 1~3 的元素
		* `views::join`
			* 扁平化嵌套容器（比如`vector<vector<int>>`→`vector<int>`）
			* 例：`vv|views::join` // 扁平化二维数组
4. Ranges 版算法
	1. 普通API函数
		* 排序（`sort`）
			```cpp
			//传统STL写法
			sort(nums.begin(),nums.end()); //升序
			sort(nums.begin(),nums.end(),greater<int>()); //降序
			//Ranges简化写法
			ranges::sort(nums); //升序
			ranges::sort(nums,greater<int>()); //降序
			```
		* 查找（`find/find_if`）
			```cpp
			//传统STL写法
			auto it1=find(nums.begin(),nums.end(),target); //找值
			auto it2=find_if(nums.begin(),nums.end(),[](int x) { return x>3; }); //找条件
			//Ranges简化写法
			auto it1=ranges::find(nums,target); //直找值
			auto it2=ranges::find_if(nums,[](int x) { return x>3; }); //找条件
			```
		* 计数（`count/count_if`）
			```cpp
			//传统STL写法
			int cnt1=count(nums.begin(),nums.end(),target); //统计目标值的个数
			int cnt2=count_if(nums.begin(),nums.end(),[](int x) { return x%2==0; }); //统计满足某个条件的个数
			//Ranges简化写法
			int cnt1=ranges::count(nums,target); //统计目标值的个数
			int cnt2=ranges::count_if(nums,[](int x) { return x%2==0; }); //统计满足某个条件的个数
			```
		* 上下界（`lower_bound/upper_bound`）
			```cpp
			//传统STL写法
			auto it1=lower_bound(nums.begin(),nums.end(),target); //≥目标值
			auto it2=upper_bound(nums.begin(),nums.end(),target); //＞目标值
			//Ranges简化写法
			auto it1=ranges::lower_bound(nums,target); //≥目标值
			auto it2=ranges::upper_bound(nums,target); //＞目标值
			//进阶：Ranges支持视图区间（传统写法需手动截迭代器）
			auto sub_view=nums|views::drop(1)|views::take(3); //子视图
			auto it3=ranges::lower_bound(sub_view,target); //直接在子视图上找
			```
		* 包含判断（`contains`（C++23））
			```cpp
			//传统STL写法
			bool exist=find(nums.begin(),nums.end(),target)!=nums.end();
			//Ranges简化写法（C++23）
			bool exist=ranges::contains(nums,2);
			```
		* 删除元素（`erase/remove_if`）
			```cpp
			//传统STL写法（erase-remove惯用法）
			nums.erase(remove_if(nums.begin(),nums.end(),[](int x) { return x%2==0; }), nums.end());
			//Ranges简化写法（C++20 erase_if）
			ranges::erase_if(nums, [](int x) { return x%2==0; });
			```
		* 遍历执行操作（`for_each`）
			```cpp
			//传统STL写法
			for_each(nums.begin(),nums.end(),[](int x) { cout << x << " "; });
			//Ranges简化写法
			ranges::for_each(nums,[](int x) { cout << x << " "; });
			//进阶：遍历视图（过滤+打印）
			ranges::for_each(nums|views::filter([](int x) { return x%2==0; }),[](int x) { cout << x << " "; });
			```
		* 替换元素（`replace/replace_if`）
			```cpp
			//传统STL写法
			replace(nums.begin(),nums.end(),a,b); //把a换成b
			replace_if(nums.begin(),nums.end(),[](int x) { return x%2==0; },b); //偶数换b
			//Ranges简化写法
			ranges::replace(nums,a,b); //把a换成b
			ranges::replace_if(nums,[](int x) { return x%2==0; },b); //偶数换b
			```
		* 反转容器（`reverse`）
			```cpp
			//传统STL写法
			reverse(nums.begin(),nums.end());
			//Ranges简化写法
			ranges::reverse(nums);
			//进阶：反转视图（不修改原容器）
			auto rev_view=nums|views::reverse;
			for (int num:rev_view) cout<<num<<" ";
			```
		* 找最大值 / 最小值（`max_element/min_element`）
			```cpp
			//传统STL写法
			auto max_it=max_element(nums.begin(),nums.end()); //找最大值迭代器
			int max_val=*max_it; //必须解引用才能拿到值
			auto min_it=min_element(nums.begin(),nums.end()); //找最小值迭代器
			int min_val=*min_it;
			//Ranges简化写法
			//方式1：找迭代器
			auto max_it=ranges::max_element(nums);
			auto min_it=ranges::min_element(nums);
			//方式2：直接取值（C++20新增，无需解引用）
			int max_val2=ranges::max(nums);
			int min_val2=ranges::min(nums);
			```
	2. 算法
		* 生成连续序列（`iota`）
			```cpp
			//传统STL写法（手动循环生成）
			vector<int> nums(5);
			for(int i=0;i<5;i++) nums[i]=i+1;
			//Ranges简化写法（视图生成，无临时容器）
			auto nums_view=views::iota(1, 6); //生成虚拟序列
			for(int num:nums_view) cout<<num<<" "; //遍历才生成
			//进阶：生成序列+转换
			auto doubled=views::iota(1,6)|views::transform([](int x) { return x*2; });
			for (int num:doubled) cout<<num<<" ";
			```
		* 过滤 + 转换（`filter+transform`）
			```cpp
			//场景：过滤偶数 → 翻倍 → 取前 3 个
			//传统STL写法（需2个临时容器，拷贝2次）
			vector<int> temp1,temp2;
			for(int num:nums) if(num%2==0) temp1.push_back(num); //过滤偶数
			for(int num:temp1) temp2.push_back(num*2); //翻倍
			for(int i=0;i<3;i++) cout<<temp2[i]<<" "; //取前3个
			//Ranges简化写法（无临时容器，管道符串联）
			auto result=nums|views::filter([](int x) { return x%2==0; })|views::transform([](int x) { return x*2; })|views::take(3);
			for (int num:result) cout<<num<<" ";
			```
		* 扁平化嵌套容器（`join`）
			```cpp
			//场景：将二维 vector 转为一维（比如 {{1,2},{3,4}} → [1,2,3,4]）
			//传统STL写法（需临时容器，手动遍历嵌套）
			vector<int> flat;
			for(auto& sub:vv){
				for(int num:sub) flat.push_back(num);
			}
			//Ranges简化写法（一行扁平化，无临时容器）
			auto flat_view=vv|views::join;
			for(int num:flat_view) cout<<num<<" ";
			//进阶：扁平化+过滤
			auto flat_odd=vv|views::join|views::filter([](int x) { return x%2==1; });
			for(int num:flat_odd) cout<<num<<" ";
			```
		* 切片操作（`slice/take/drop`）
			```cpp
			//场景：取容器的子区间（比如索引 1~3 的元素）
			//传统STL写法（手动计算迭代器）
			auto start=nums.begin()+1;
			auto end=nums.begin()+4;
			for(auto it=start;it!=end;it++) cout<<*it<<" ";
			//Ranges简化写法（slice视图，直接指定索引）
			auto slice_view=nums|views::slice(1,4); // 索引[1,4)
			for(int num:slice_view) cout<<num<<" ";
			//替代方案：drop+take（更灵活）
			auto slice_view=nums|views::drop(1)|views::take(3); //跳过1个，取3个
			for(int num:slice_view) cout<<num<<" ";
			```
5. 关于命名空间
	* 方式 
		```cpp
		using namespace std::views;
		using namespace std::ranges;
		```
	* 注意点
		1. 如果代码量较大，可能会有命名冲突风险。**更推荐在函数内局部引入**，仅在需要的作用域生效（**不推荐全局引入！**）
			```cpp
			int main(){
				// 局部引入views命名空间，仅在main函数内生效
				using namespace std::views;
			}
			```
		2. 可以用 `using` 声明**精准引入需要的适配器**（单个函数），避免命名冲突
			```cpp
			using std::views::filter; //精准引入views的filter
			using std::ranges::sort; //精准引入ranges的sort
			using std::ranges::max; //精准引入ranges的max
			```
		3. 对于和传统`std`命名空间下的同名函数完全重合（如`sort`、`find`、`max`、`min`），多数编译器会优先选`ranges`版，**但不推荐依赖这种行为**。若全局引入，优先使用 Ranges 版写法（如`sort(nums)`），避免传统迭代器写法