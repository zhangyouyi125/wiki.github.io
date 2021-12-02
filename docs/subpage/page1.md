```c++
#include<stdio.h>
#include<iostream>
#include<string.h>
#include<math.h>
#include<malloc.h>
#include<vector>
#include<stack>
#include <time.h>
using namespace std;
int main(){
	int n = 6, m = 8;
	//cin >> n >> m;
	vector<int> dp(m+1, 0);
	dp[0] = 1;
	for(int i = 1; i <= n; i++){
		for(int j = m; j >= i; j--){
			dp[j] = dp[j - i];
		}
	}
	cout << dp[m] <<endl;
	stack<char> st;
	cout<<st.empty()<<endl;
	cout<<"out"<<endl;
	return 0;

}


```