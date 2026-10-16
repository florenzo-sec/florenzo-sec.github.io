# florenzo-sec.github.io

Blog personale su sicurezza informatica, reti e HackTheBox. Costruito con [Hugo](https://gohugo.io/) e il tema [PaperMod](https://github.com/adityatelange/hugo-PaperMod) (MIT, incluso in `themes/PaperMod`), pubblicato con GitHub Pages tramite GitHub Actions.

## Scrivere un nuovo post

```bash
hugo new content posts/htb-nome-macchina.md
```

Modifica il file in `content/posts/`, poi `git push` su `main`: il workflow compila e pubblica il sito.

## Anteprima in locale

```bash
hugo server -D
```

## Regole di pubblicazione

- Mai la flag, mai l'IP del target o della VPN negli screenshot.
- Controllare la policy HackTheBox sui writeup prima di pubblicare.
- Gli appunti grezzi vanno in un repository privato, non qui.
