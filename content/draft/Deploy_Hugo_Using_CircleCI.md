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
- 嘗試用 CircleCI 2.0 workflow 中的 filter，屢次失敗 
- Github 認證沒有過，無法 push 到 submodule 當中
- 因為版本不對沒有辦法 push 到 submodule
- ...

因為要從 CircleCI 部署網站，撰寫文章的流程需要有所修改，我目前覺得最舒服而且相對比較簡單的方式如下：

1. 開另外一個 branch，取名叫做 develop，所有文章撰寫，或是對針對主題、版面的修改，都在這個 branch 上面完成，也都把所有的 changes 都 push 到這個 branch 當中。
2. 改好之後，確定要發布了，直接從 Github 上操作，在 master branch 中，建立 Pull Request，從 branch develop merge 文章到 master branch；當然也可以利用 Github CLI，切換到 master branch，從 develop branch merge 到 master，再 push 上去。
3. 這時候因為有寫 CircleCI 設定檔，Github 會把這個 merge 推送到 CircleCI，CircleCI 針對這個 commit 做一些檢查，如果沒有問題的話，會執行 `hugo` 指令，建立靜態網站，並且把 `public` 資料夾下的東西推送到 `<your-account>.github.io` repo，自動部署網站。

那到底要怎麼完成呢？

### 環境設定
我的環境如下：

- Hugo 文章都存在 repo `github_io_content` 中
- Hugo 網站放在 repo `<your-account.github.io>` 中

步驟如下：

1. 在 CircleCI 開帳號，並且設定好我們要 build 的 project 是 `github_io_content`，也就是你放所有的文章的那個 projecy，**不是** 你部署上去的那個。
2. 其實 CircleCI 會自動建立 key，不過是 read only 的，所以我們要建立另外一把 key [(建 key 教學)][github_key_generation]，專門給 CircleCI 用的，讓環境乾淨一點，這把 key 要可讀可寫。
3. 把這把 key 的 public key 加到 Github repo `<your-account.github.io>` 的 deploy key 中。[參考文件][circleci_readwrite_key]
4. 把這把 key 的 private key 加到 CircleCI project `github_io_content` 中。[參考文件][circleci_readwrite_key]
5. 寫 CircleCI 專屬的 `config.yaml` 檔於 `.circleci` 資料夾中，

config.yaml 內容如下，下面有更詳細介紹。
```
version: 2
global: &global
  working_directory: ~/project
  docker:
    - image: felicianotech/docker-hugo:0.30.2

jobs:
  build_n_deploy:
    <<: *global
    steps:
      - checkout
      - run:
          name: "Run Hugo"
          command: |
            git submodule sync && git submodule update --init 
            git submodule foreach --recursive git pull origin master
            HUGO_ENV=production hugo -v
      
#      - run:
#          name: "Test Website"
#          command: htmlproofer ~/project/public --allow-hash-href --check-html --empty-alt-ignore --disable-external --checks-to-ignore "HtmlCheck"

      - add_ssh_keys:
          fingerprints:
            - "44:4d:07:a8:35:...fingerprint_of_key"
      - run:
          name: "Deploy to github.io"       
          command: |
            cd ~/project/public
            git config --global user.email "you@email.com"
            git config --global user.name "First Last"
            git add -A
            git commit -m "Deploy from CircleCI"
            git push origin HEAD:master

workflows:
  version: 2
  build_n_deploy:
    jobs:
      - build_n_deploy:
          filters:
            branches:
              only: master

```

### 設定檔解釋

參考資料:

[github_key_generation]: https://help.github.com/articles/connecting-to-github-with-ssh/
[circleci_readwrite_key]: https://circleci.com/docs/1.0/adding-read-write-deployment-key/
[felicianotech-docker(git)]: https://github.com/felicianotech/docker-hugo
[deploy_guide_circleCI_post]: https://circleci.com/blog/build-test-deploy-hugo-sites/
[additional_circleCI_post]: https://circleci.com/blog/circleci-hacks-reuse-yaml-in-your-circleci-config-with-yaml/