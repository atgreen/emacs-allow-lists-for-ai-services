# Allow-lists for AI service-assist in Emacs

Emacs has some useful integrations with AI services like OpenAI's ChatGPT and GitHub's co-pilot.  However, I don't want to enable those services globally.  With the following elisp fragment, I can choose to allow AI service-assist in specific git repos by either creating an `.allow-ai-service` file in the top-level directory of my project, or adding the top-level directory to a list in `~/.allow-ai-service-list`.

```
(defun my/wrap-ai-service (wrapped-fn &rest args)
  "Only call WRAPPED-FN when the current project is on the AI
 service allow-list."
  (if (let ((root-dir (locate-dominating-file "." ".git")))
        (when root-dir
          (let ((allow-file (expand-file-name ".allow-ai-service" root-dir))
                (allow-list-file (with-temp-buffer
                                   (insert-file-contents (expand-file-name "~/.allow-ai-service-list"))
                                   (split-string (buffer-string) "\n" t))))
            (or (file-exists-p allow-file)
                (member root-dir allow-list-file)))))
        (apply wrapped-fn args)))
```

Now we simply use emacs' [advice feature](https://www.gnu.org/software/emacs/manual/html_node/elisp/Advising-Functions.html) to wrap the functions we want to check.  Wrapped functions will only be called if they are explicitly allowed.   For instance, if you are using [gpt-commit](https://github.com/ywkim/gpt-commit), add:
```
(advice-add 'gpt-commit-message :around #'my/wrap-ai-service)
```

Or if you are using [copilot-mode](https://github.com/copilot-emacs/copilot.el), do this:
```
(advice-add 'copilot-mode :around #'my/wrap-ai-service)
```

Your `~/.allow-ai-service-list` file should just be a list of abbreviated (using `~`) directory names (ending in `/`).  For example:
```
~/my-project/
~/my-other-project/
```

Happy hacking!

Anthony Green
