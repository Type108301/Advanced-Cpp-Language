
1. 从赋值说起 
	C++中有**将 “可调用对象”（比如lambda、函数指针、函数对象）赋值给变量**的写法
	1. **函数指针**
		```cpp
		//定义普通函数 
		int add(int a,int b){
			return a+b;
		}
		int main(){
			//定义函数指针变量，赋值为add函数的地址 
			//格式返回值类型 (*变量名)(参数类型列表)=函数名;
			int (*func_ptr)(int, int)=add;
			//调用这个变量（和调用函数一样）
			cout<<func_ptr(3, 5)<<endl; //输出8
			return 0;
		}
		```
	2. 用` auto` 简化函数指针（C++11 及以上）
		```cpp
		int add(int a,int b){
			return a+b;
		}
		int main(){
			//auto自动推导func的类型为“函数指针int (*)(int,int)”
			auto func=add;
			cout<<func(3,5)<<endl; //输出8
			return 0;
		}
		```
	3. **lambda**
		* 基础 lambda 赋值（无递归）
			```cpp
			int main(){
				//把lambda赋值给变量print_num
				auto print_num=[](int num){
					cout<<num<<endl;
				};
				//调用变量（和调用函数一样）
				print_num(10); //输出:10
				return 0;
			}
			```
		* 递归 lambda 赋值（C++23）
			```cpp
			int main(){
				//递归lambda：把lambda赋值给dfs变量
				auto dfs=[&](this auto&& dfs,int i){
					if(i==0) return;
					cout<<i<<" ";
					dfs(i-1); //调用自身（通过参数dfs）
				};
				dfs(5); //输出:5 4 3 2 1 
				return 0;
			}
			```
			* 传统写法中 lambda 无法直接引用自身（因为定义时还没有名字），而 C++23 的`this`关键字解决了这个问题：
				1. `this`：声明 lambda 的 “自引用”，允许参数中的`dfs`指代 lambda 自身；
				2. `auto&& dfs`：泛型参数，接收 lambda 自身的右值引用（支持完美转发）； 
			* 如果不用`this`关键字，则需要引入`std::function`
				```cpp
				int main(){
					//先声明std::function类型
					function<void(int)> dfs;
					//再赋值lambda，捕获dfs的引用
					dfs=[&](int i){
						if(i==0) return;
						cout<<i<<" ";
						dfs(i-1); //借助std::function调用自身
					};
					dfs(5); //输出:5 4 3 2 1 
					return 0;
				}
				```
	4. **仿函数**（重载 operator () 的类对象）
		```cpp
		//定义仿函数类：重载operator()
		class AddFunc{
		public:
			int operator()(int a,int b){
				return a+b;
			}
		};
		int main(){
			//把仿函数对象赋值给变量add_obj
			AddFunc add_obj;
			//调用变量（等价于调用operator()）
			cout<<add_obj(3,5)<<endl; //输出8
			return 0;
		}
		```
2. 进入正题
	lambda 本质是C++11引入的**匿名函数对象**
	1. 完整语法模板
		```cpp
		[捕获列表](参数列表) -> 返回值类型 { 函数体 }
		```
	2. 举例与解释
		用排序函数拆解：
		```cpp
		[](int a,int b){ return abs(a)>abs(b); }
		```
		|部分|示例中的内容|解释|
		|---|---|---|
		|捕获列表 `[]`|`[]`|决定 lambda 能否用周围的变量（比如 main 里的变量）；空`[]`表示 “不用任何外部变量”|
		|参数列表 `()`|`(int a,int b)`|和普通函数的参数一样，是 lambda 要接收的输入|
		|返回值类型 `-> 类型`|省略了|C++11 及以上会自动推导返回值（这里返回 bool），简单场景可省略|
		|函数体 `{}`|`{ return abs(a)>abs(b); }`|lambda 要执行的逻辑（和普通函数的函数体一样）|
	3. 捕获列表
		捕获列表决定 lambda 能否访问 **“定义它的作用域里的变量”**
		
		|捕获方式|示例|解释|
		|---|---|---|
		|`[&]`|`[&](int x) { nums.push_back(x); }`|按**引用**捕获所有外部变量（能读能改）|
		|`[=]`|`[=]() { cout << nums.size(); }`|按**值**捕获所有外部变量（只能读，不能改）|
	4. 使用方式（两种）
		* 直接用（匿名调用）
			直接把 lambda 写在需要的地方（比如`sort`的第三个参数），不用赋值给变量，用完就扔。
		* 赋值给变量（复用）
			如果 lambda 要多次调用，就赋值给变量
	5. 注意要点
		* **lambda 的生命周期**：lambda 的捕获列表如果捕获了局部变量的引用（`[&]`），当局部变量销毁后，lambda 再调用会崩溃（比如把 lambda 返回出去，外部变量已经没了）；
		* **const 限制**：`[=]`捕获的变量是 const 的，不能修改；如果想改，要加`mutable`：
			```cpp
			auto func=[=]() mutable { add_num=20; }; //加mutable才能改值捕获的变量
			```
		* **参数和返回值**：lambda 的参数列表可以为空（`()`），返回值能自动推导，简单场景不用写`-> 类型`
