## github上传
echo "# job-note" >> README.md  
git init  
git add README.md  
git commit -m "first commit"  
git branch -M main  
git remote add origin https://github.com/SEUsyx/job-note.git  
git push -u origin main  

首次推送：
git remote add origin https://github.com/SEUsyx/job-note.git  
git branch -M main  
git push -u origin main


## git操作流程

- git add .     将所有修改添加到暂存区
- git commit -m "描述"      将更改记录提交到本地存储库
- git push      将更改记录提交到远程库