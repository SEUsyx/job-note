# kmp算法

```C++
#include<iostream>
#include<vector>

using namespace std;

vector<int> getNext(const string& substr){
    vector<int> next(substr.size(),-1);
    int j=-1;
    for(int i=1; i < substr.size(); i++){
        while(j>=0 && substr[i]!=substr[j+1]){
            j=next[j];
        }
        if(substr[i]==substr[j+1]){
            j++;
        }
        next[i]=j;
    }
    return next;
}

int KMP(const string& str, const string& substr){
    vector<int> next= getNext(substr);
    int j=-1;
    for(int i=0; i<str.size();i++){
        while(j>=0 && str[i]!=substr[j+1]){
            j=next[j];
        }
        if(str[i]==substr[j+1]){
            j++;
        }
        if(j==substr.size()-1) return i-substr.size()+1;
    }
    return -1;
}

int main() {
    cout<<KMP("aabaabaaf","aabaaf")<<endl;
    return 0;
}


```