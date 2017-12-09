---
title: "使用 CircleCI 部署 Hugo 靜態網站"
date: 2017-12-09T09:51:03+08:00
draft: true
---
搞了半天，終於可以利用 CircleCI 來部署 Hugo 靜態網站到 Github Page，這當中出現出現非常多的錯誤，累積失敗的 Build 高達七十個，把這些錯誤的例子都寫寫，以後如果有人看到或是自己想要重新 Reference 都比較方便。

因為沒有裝 CircleCI CLI，很多 commit 都只改一行 `config.yaml`，還是建議可以裝一下 CLI，才不會紅紅一片看了不爽快，我遇到的錯誤如下：

- 在 CircleCI 上面沒辦法正常產生靜態網站，錯誤訊息: `nothing to commit, working directory clean` 
- 同樣的 commit，但是卻出現重複的 build，一個成功，一個失敗
- 嘗試採用 CircleCI 2.0 的 workflow，不過無法成功部署
- Github 認證沒有過，無法 push 到 submodule 當中
- ...


參考資料:

[felicianotech-docker(git)]: https://github.com/felicianotech/docker-hugo
[deploy_guide_circleCI_post]: https://circleci.com/blog/build-test-deploy-hugo-sites/
[additional_circleCI_post]: https://circleci.com/blog/circleci-hacks-reuse-yaml-in-your-circleci-config-with-yaml/