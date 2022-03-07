# nSum问题


```C++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

vector<vector<int>> twoSum(const vector<int> & nums, int start, int target){ //2sum模板（不包含重复数字），双指针
    int left=start, right= nums.size() - 1;
    vector<vector<int>> result;
    while(left < right){
        int lnum=nums[left],rnum=nums[right];
        int sum= nums[left] + nums[right];
        if(sum<target){
            while(left < right && nums[left] == lnum) left++; //去重
        }
        else if(sum>target){
            while(left < right && nums[right] == rnum) right--; //去重
        }
        else{
            result.push_back({lnum,rnum});
            while(left < right && nums[left] == lnum) left++; //去重
            while(left < right && nums[right] == rnum) right--; //去重
        }

    }
    return result;
}

vector<vector<int>> nSum(const vector<int>& nums, int n, int start, int target){
    vector<vector<int>> result;
    if(n<2 || nums.size()-start<n) {
        return result;
    }
    if(n==2){
        return twoSum(nums,start,target);
    }
    else{
        int flag=start;
        while(flag<nums.size()){
            int curnum=nums[flag];
            vector<vector<int>> tmp=nSum(nums,n-1,flag+1,target-curnum); //递归调用
            if(!tmp.empty()){
                for(auto& vec:tmp){
                    vec.push_back(curnum);
                    result.push_back(vec);
                }
            }
            while(flag<nums.size() && nums[flag]==curnum) flag++;  //nums[flag]去重
            //因为curnum=nums[flag]，一定会往前走一步的，所以省略了循环外flag++
        }

    }
    return result;
}


int main() {
    vector<int> nums({-1,0,1,-1});
    sort(nums.begin(), nums.end());  // 一定要先排序再使用
    vector<vector<int>> result=nSum(nums,3,0,0);
    
    for(auto vec:result){
        for(auto x:vec){
            cout<<x<<",";
        }
        cout<<endl;
    }

    return 0;
}
```