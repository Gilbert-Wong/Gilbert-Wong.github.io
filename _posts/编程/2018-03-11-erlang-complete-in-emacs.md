---
layout: blog
istop: true
background-image: /images/2018-03-11-erlang-complete-in-emacs-1.png
title: emacs 上的 Erlang 自动补全
date: 2018-03-12
category: 编程
tags:
- erlang
- company-erlang
- ivy-erlang-complete
- emacs 
---

Erlang 虽然在　Emacs 中有对应的 mode，可以在 emacs 中灵活地编译执行 erlang 代码，但是默认的 erlang-mode 还是有很多不足的地方。这里我介绍两个用于增强 Erlang 编程的两个扩展。 分别是 `ivy-erlang-complete` 和 `company-erlang`


ivy-erlang-complete 的功能非常强大， 除了能够通过函数 `ivy-erlang-complete()` 来实现 erlang 代码的自动补全，还能查找变量的定义和引用，甚至直接在浏览器中直接打开 erlang-otp 的相关文档。 如果你是 emacs 新手，且无意于使用他人集成好的 emacs 配置，你可以参考 `ivy-erlang-complete` 的作者 s-kostyaev　的 emacs 配置，当然我更建议在 spacemacs 的基础上去使用和定制这款扩展。

s-kostyaev 的配置可以直接看他的 github 仓库: [s-kostyaev/ivy-erlang-complete](https://github.com/s-kostyaev/ivy-erlang-complete) 以及　[s-kostyaev/company-erlang](https://github.com/s-kostyaev/company-erlang) ，还有 [s-kostyaev/.emacs.d](https://github.com/s-kostyaev/.emacs.d)。

我使用的是 spacemacs ， 这里我把配置写在自己的 layer 里的。

```elisp
(setq gilbert-programming-packages
      '(
        paredit
        ivy-erlang-complete
        company-erlang
        ))


;; enhance erlang-mode
(defun gilbert-programming/init-ivy-erlang-complete()
  "Setup for erlang."
  (use-package ivy-erlang-complete
    :defer t
    :ensure t
    :config
    (progn
      (add-hook 'erlang-mode-hook #'company-erlang-init)
      (add-hook 'erlang-mode-hook #'ivy-erlang-complete-init)
      ;; automatic update completion data after save
      (add-hook 'after-save-hook #'ivy-erlang-complete-reparse)
      (add-to-list 'auto-mode-alist '("rebar\\.config$" . erlang-mode))
      (add-to-list 'auto-mode-alist '("relx\\.config$" . erlang-mode))
      (add-to-list 'auto-mode-alist '("system\\.config$" . erlang-mode))
      (add-to-list 'auto-mode-alist '("\\.app\\.src$" . erlang-mode))
      ;; (dolist (prefix '(
      ;;                   ("e" . "erlang-ide")
      ;;                   ))
        ;; (spacemacs/declare-prefix-for-mode
        ;;   'erlang-mode (car prefix) (cdr prefix)))

      (spacemacs/set-leader-keys-for-major-mode 'erlang-mode
        "d" 'ivy-erlang-complete-find-definition
        "s" 'ivy-erlang-complete-find-spec
        "r" 'ivy-erlang-complete-find-references
        "f" 'ivy-erlang-complete-find-file
        "h" 'ivy-erlang-complete-show-doc-at-point
        "p" 'ivy-erlang-complete-set-project-root
        "a" 'ivy-erlang-complete-autosetup-project-root
        ))

    :custom
    (ivy-erlang-complete-erlang-root "/usr/local/Cellar/erlang/20.2.4/lib/erlang")
    )
  )

(defun gilbert-programming/init-company-erlang()
  (add-hook 'erlang-mode-hook #'company-erlang-init)
  )
```
![](/images/2018-03-11-erlang-complete-in-emacs-1.png)

![](/images/2018-03-11-erlang-complete-in-emacs-2.png)

![](/images/2018-03-11-erlang-complete-in-emacs-3.png)


 注意：
 
 如果你是 mac 用户，请使用 homebrew  `brew uninstall sed` 和 `brew install gnu-sed --with-default-names`。　 `ivy-erlang-complete` 的包依赖的是 `gnu-sed` 而不是 Mac 自带的 `bsd-sed`。　 homebrew 默认安装的 sed 名称是 gsed。这里设定参数重新安装一下 sed 。同时在自己的 shell 配置里设定一下 `gnu-sed`  的 path 就没问题了。
