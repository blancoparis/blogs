# iterm2 (Mac)

## Oh my zsh

Se instala con el siguiente comando

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### autosuggestions

```bash
brew install zsh-autosuggestions
```

```bash
echo "source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh" >> .zshrc
```

### completions

```bash
brew install zsh-completions
```

En el fichero **~/.zshrc**, al final ponemos

```bash
if type brew &>/dev/null; then
  FPATH=$(brew --prefix)/share/zsh-completions:$FPATH
 
  autoload -Uz compinit
  compinit -u
fi
```




## Tipologia

https://github.com/romkatv/powerlevel10k

1. Instalamos las letras
Download these four ttf files:
MesloLGS NF Regular.ttf
MesloLGS NF Bold.ttf
MesloLGS NF Italic.ttf
MesloLGS NF Bold Italic.tt
> Hacemos click en cada uno de los archivos.

2.  
