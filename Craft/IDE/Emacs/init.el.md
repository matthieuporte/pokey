
This is our emacs config file.

>[!info]- Tutorials
> [rtags1](https://martinsosic.com/development/emacs/2017/12/09/emacs-cpp-ide.html#rtags) - [rtags2](https://trivialfis.github.io/emacs/2017/08/02/C-C++-Development-Environment-on-Emacs.html) - [lsp setup](http://tuhdo.github.io/c-ide.html)

>[!danger]- Inspiration

---

```elisp
;;
;; clement.pochart and matthiru's
;;
;; o __  o _|_    _  |
;; | | | |  |_ o (/_ |
;;


;; SENSIBLE DEFAULTS
;; -----------------

(setq inhibit-startup-message t)  ; no startup message
(scroll-bar-mode -1)              ; disable the visible scrollbar
(menu-bar-mode -1)                ; disable the menu-bar
(tool-bar-mode -1)                ; disable the toolbar
(tooltip-mode -1)                 ; disable the tooltips
(set-fringe-mode 5)               ; adds some margins
(setq visible-bell t)             ; Visually show errors instead of beeping
(setq gc-cons-threshold 50000000) ; Reduces the freq. of garbage collection
 ; Displays the numbers to the left
(add-hook 'prog-mode-hook 'display-line-numbers-mode)
;; (setq display-line-numbers-type 'relative)


;; AUTO-SAVE FILES
;; ---------------

(setq backup-directory-alist `(("." . "~/.emacs.d/autosaves")))
;; (setq backup-directory-alist `(("." . "~/.emacs-autosaves")))

;; PACKAGE MANAGEMENT
;; ------------------

; enable the "package" package for package management
(require 'package)
(setq package-archives '(("gnu" . "https://elpa.gnu.org/packages/")
						 ("melpa" . "https://melpa.org/packages/")
                         ("org" . "https://orgmode.org/elpa/")))

(package-initialize)
;; if there is no package archive locally, download it.
(unless package-archive-contents
  (package-refresh-contents))

; Install the package "use-package" if not installed yet
(unless (package-installed-p 'use-package)
  (package-install 'use-package))

(require 'use-package)

; Installs all the packages
(setq use-package-always-ensure t)

;; EVIL MODE
;; ---------

(use-package evil
  :diminish
  :init
  (setq evil-want-integration t)
  (setq evil-want-keybinding nil)
  (setq evil-want-C-u-scroll t)
  (setq evil-undo-system 'undo-redo)
  :config
  (evil-mode 1))

(use-package evil-collection
  :after evil
  :config
  (evil-collection-init))

(use-package evil-commentary
  :config (evil-commentary-mode))

(evil-define-key 'normal 'global (kbd "C-u") (lambda () (interactive) (evil-scroll-up nil) (evil-scroll-line-to-center (line-number-at-pos))))
(evil-define-key 'normal 'global (kbd "C-d") (lambda () (interactive) (evil-scroll-down nil) (evil-scroll-line-to-center (line-number-at-pos))))

(use-package evil-numbers
  :config
  (global-set-key (kbd "C-=") 'evil-numbers/inc-at-pt)
  (global-set-key (kbd "C--") 'evil-numbers/dec-at-pt))

;; FONT AND THEME
;; --------------

(set-face-attribute 'default nil :font "Inconsolata" :height 120)
;; (set-face-attribute 'default nil :font "Mononoki Nerd Font Mono" :height 140)
(use-package zenburn-theme
  :ensure t
  :config (load-theme 'zenburn t))
;; (use-package kaolin-themes
;;   :config (load-theme 'kaolin-dark t))

;; MINIBUFFER ENHANCEMENTS
;; -----------------------

(use-package ivy
  :diminish
  :bind (("C-s" . swiper)
         :map ivy-minibuffer-map
         ("TAB" . ivy-alt-done)	
         ("C-l" . ivy-alt-done)
         ("C-j" . ivy-next-line)
         ("C-k" . ivy-previous-line)
         :map ivy-switch-buffer-map
         ("C-k" . ivy-previous-line)
         ("C-l" . ivy-done)
         ("C-d" . ivy-switch-buffer-kill)
         :map ivy-reverse-i-search-map
         ("C-k" . ivy-previous-line)
         ("C-d" . ivy-reverse-i-search-kill))
  :config
  (ivy-mode 1))

(when (bound-and-true-p evil-mode)
  (define-key evil-normal-state-map (kbd "/") 'swiper))

(use-package counsel)

;; PROGRAMMING
;; -----------

(use-package avy)

(use-package which-key)

(use-package clang-format
  :commands (clang-format-buffer clang-format-on-save-mode)
  :hook (c-mode c++-mode)
  :config
  (clang-format-on-save-mode))

(setq c-default-style '((c-mode . "awk") ;; default style close from
                        (c++-mode . "awk") ;; the coding style
                        (other . "awk")))


;; DIRED
;; -----

(defun my-dired-quit-and-kill-buffers ()
  "Close all Dired buffers and quit Dired mode."
  (interactive)
  (mapc (lambda (buffer)
          (when (eq (buffer-local-value 'major-mode buffer) 'dired-mode)
            (kill-buffer buffer)))
        (buffer-list))
  (message "Closed all Dired buffers"))

(with-eval-after-load 'dired
  (evil-define-key 'normal dired-mode-map
    "h" 'dired-up-directory
    "l" 'dired-find-file
    "q" 'my-dired-quit-and-kill-buffers
    " " 'nil))

;; PROJECTILE
;; ----------

(use-package projectile
  :init
  (setq projectile-project-search-path '("~/projects/" "~/work/" "~/playground"))
  :config
  (global-set-key (kbd "C-c p") 'projectile-command-map)
  (projectile-mode +1))

(use-package counsel-projectile
 :after projectile
 :config
 (counsel-projectile-mode 1))

;; GENERAL AND HYDRA
;; -----------------

(use-package general
  :config
  (general-evil-setup t)

  (general-create-definer rune/leader-keys
    :keymaps '(normal insert visual emacs)
    :prefix "SPC"
    :global-prefix "C-SPC"))


(defun copy-whole-buffer ()
  "Copy the entire buffer to the kill ring."
  (interactive)
  (kill-ring-save (point-min) (point-max)))


(rune/leader-keys
  "s"  '(swiper :which-key "swiper"))

(rune/leader-keys
  "d"  '(lambda () (interactive) (dired default-directory) :which-key "dired"))

(rune/leader-keys
  "e"  '(:ignore t :which-key "execute")
  "ej" '(counsel-M-x :which-key "M-x"))

(rune/leader-keys
  "t"  '(:ignore t :which-key "toggles")
  "ts" '(hydra-text-scale/body :which-key "scale text")
  "ta" '(hydra-all-switch-theme/body :which-key "toggle ALL themes")
  "tt" '(hydra-switch-theme/body :which-key "toggle favorite theme"))

(rune/leader-keys
  "f"  '(:ignore t :which-key "find")
  "ff" '(find-file :which-key "find file")
  "fs" '(save-buffer :which-key "file save")
  "fh" '((lambda () (interactive) (switch-to-buffer "*scratch*")) :which-key "scratch")
  "fi" '((lambda() (interactive)(find-file "~/.emacs.d/init.el")) :which-key "init.el"))

(rune/leader-keys
  "j"  '(:ignore t :which-key "jump")
  "jl"  '(avy-goto-line :which-key "avy go to line")
  "ja"  '(avy-goto-word-1 :which-key "avy go to word 1"))

(rune/leader-keys
  "b"  '(:ignore t :which-key "buffer")
  "bs" '(hydra-buffer-switch/body :which-key "buffer switch")
  "by" '(copy-whole-buffer :which-key "yank buffer")
  "bl" '(ibuffer :which-key "buffer list")
  "be" '(eval-buffer :which-key "eval buffer")
  "bj" '(next-buffer :which-key "next")
  "bd" '(kill-this-buffer :which-key "kill buffer")
  "bk" '(previous-buffer :which-key "previous"))

(rune/leader-keys
  "p"  '(:ignore t :which-key "projectile")
  "pp" '(projectile-switch-project :which-key "switch project")
  "pa" '(projectile-find-other-file :which-key "find other file")
  "pg" '(counsel-projectile-grep :which-key "grep")
  "pf" '(projectile-find-file :which-key "find file in project"))

(rune/leader-keys
  "w"  '(:ignore t :which-key "windows")
  "wv" '(split-window-horizontally :which-key "split window vertically")
  "wh" '(split-window-vertically :which-key "split window horizontally")
  "wd" '(delete-window :which-keh "close current window"))


```
