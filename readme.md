# BLOCO 1 — setup inicial
1. Abra o terminal na pasta onde quer guardar o projeto
2. Crie e entre no diretório
```bash
mkdir oficina-git-lab
cd oficina-git-lab
```
3. Inicie o repositório
```bash
git init
echo "# Oficina Avançada de Git" > README.md
git add README.md
git commit -m "init: projeto da oficina"
```
4. Confirme
```bash
git log --oneline
```














# BLOCO 2 — Criar commits bagunçados para treinar `git rebase -i`
1. Crie um arquivo app.txt e vá alterando aos poucos

Criação do arquivo inicial
```bash
echo "Sistema de Login" > app.txt
git add app.txt
git commit -m "feat: cria estrutura base"
```
```bash
echo "Sistema de Login" > app.txt
git add app.txt
git commit -m "feat: cria estrutura base"
```
Adicione um título
```bash
echo "Título: Login de Usuário" >> app.txt
git add app.txt
git commit -m "fix: adiciona título"
```
Corrija o título
```bash
sed -i 's/Login de Usuário/Autenticação de Usuário/' app.txt
git add app.txt
git commit -m "chore: corrige nome do título"

```
Adicione validação
```bash
echo "Valida campos de e-mail e senha." >> app.txt
git add app.txt
git commit -m "feat: validações de campos"
```
2. Verifique a linha do tempo
```bash
git log --oneline

```
Você deve ver algo como:
```yaml
a1b2c3d (HEAD -> main) feat: validações de campos
1234567 chore: corrige nome do título
89abcd0 fix: adiciona título
0abc123 feat: cria estrutura base
xyz0000 init: projeto da oficina
```
## BLOCO 2.1 -- Reorganizando commits com `git rebase -i`
1. Rode o rebase interativo
```bash
git rebase -i HEAD~4
```
2. Limpe o histórico conforme este plano:
Na tela do rebase, você verá algo como:
```yaml
pick aaaaaaa feat: cria estrutura base
pick bbbbbbb fix: adiciona título
pick ccccccc chore: corrige nome do título
pick ddddddd feat: validações de campos
```
Agora altere para:
```yaml
pick aaaaaaa feat: cria estrutura base
squash bbbbbbb fix: adiciona título
squash ccccccc chore: corrige nome do título
squash ddddddd feat: validações de campos
```
ou  use 'r' para reword o primeiro e 's' para squash os seguintes
```yaml
r aaaaaaa feat: cria estrutura base
s bbbbbbb fix: adiciona título
s ccccccc chore: corrige nome do título
s ddddddd feat: validações de campos
```
3. Escreva a nova mensagem do commit

Aparecerá uma tela para editar a mensagem do commit “squashado”.
Você pode limpar tudo e deixar algo como:
```
feat: cria estrutura base do login com título e validações
```
4. Confirme que o histórico foi limpo
```bash
git log --oneline
```
Você verá:
```bash
<hash> feat: cria estrutura base do login com título e validações
<hash> init: projeto da oficina
```













# BLOCO 3 — Branch `feature/login` × `main` (conflito na prática)
## 3.1 – Crie e trabalhe em `feature/login`
1. Crie a branch e altere o arquivo
```bash
git checkout -b feature/login
```
Edite app.txt (linha final) para mostrar a frase abaixo:
```text
echo "Valida campos de e-mail e senha. Exibe mensagem “Bem-vindo” após login." >> app.txt
```
Depois:
```bash
git add app.txt
git commit -m "feat(login): exibe mensagem de boas-vindas"
```
2. Faça mais uma pequena melhoria
```bash
echo "Registra tentativas de login no log de auditoria." >> app.txt
git add app.txt
git commit -m "feat(login): log de auditoria"
```
## 3.2 – Volte à main e crie uma mudança que conflita
```bash
git checkout main
```
Abra app.txt e modifique a mesma linha que começava com “Valida campos…”.
Troque por:
```text
Valida campos de e-mail e senha. BLOQUEIA IP após 5 tentativas.
```
```bash
git add app.txt
git commit -m "feat(security): bloqueio de IP após 5 tentativas"
```
Agora a main e a feature/login divergem na mesma linha ⇒ conflito garantido.
## 3.3 – Tente mesclar (provocar o conflito)
```bash
git merge feature/login
```
Você verá um output indicando CONFLICT (content) em app.txt.

Vamos resolver dentro do próprio VS Code (sem sair para o terminal) e depois concluir o merge.

1. Finalize o merge no Source Control (Git) panel
- No canto esquerdo (ícone de ramo), o VS Code mostra “Merge Changes” e um botão ✓ Commit.
- Escreva uma mensagem como:
    ```bash
    merge: resolve conflito entre main e feature/login
    ```
- Clique ✓ ou pressione Ctrl + Enter.
2. Verifique no terminal (opcional)
```bash
git status
git log --oneline --graph
```
Confirme que o merge commit aparece no topo
















# BLOCO 4 — Bug + `git bisect`
> Meta: colocar um teste simples que inicialmente passa, depois falha; e usar git bisect para
> identificar o commit “bugado”.
## 4.1 – Adicionar um teste que PASSA
1. Crie a pasta de testes e o script
```bash
mkdir -p tests

cat > tests/test.sh <<'EOF'
#!/usr/bin/env bash
# O teste simplesmente verifica se a palavra BUG NÃO aparece em app.txt
if grep -q 'BUG' app.txt; then
    echo -e "🔴 Teste FALHOU: BUG encontrado em app.txt"
    exit 1
else
    echo -e "🟢 Teste PASSOU"
    exit 0
fi
EOF

chmod +x tests/test.sh
```
2. Faça commit
```bash
git add tests/test.sh
git commit -m "test: adiciona teste inicial que passa"
```
3. Execute o teste para provar que passa
```bash
./tests/test.sh
```
## 4.2 – Introduzir o BUG
```bash
echo "Exibe menu superior" >> app.txt
git add app.txt
git commit -m "feat(ui): exibir menu superior"

echo "Permite que o usuário edite o perfil" >> app.txt
git add app.txt
git commit -m "feat(app): permitir editar perfil do usuário"

echo "Permite BUG alteração de senha" >> app.txt
```
Execute o teste de novo:
```bash
./tests/test.sh
```
Nesse momento você poderia fazer a correção e depois fazer o commit sem o bug, mas agora nós vamos simular uma situação onde o desenvolvedor não rodou o teste, então ele não percebeu o bug.
```bash
git add app.txt
git commit -m "feat(app): permitir alterar senha do usuário (BUG)" 
```

Perfeito: agora temos um commit ruim.

Vamos continuar adicionando mais commits
```bash
echo "Exibe botão de sair" >> app.txt
git add app.txt
git commit -m "feat(ui): exibir botão de sair"

echo "Organiza ordem dos componentes da tela de login." >> app.txt
git add app.txt
git commit -m "refactor(ui): reorganiza tela de login"

echo "# este é um comentário interno" >> app.txt
git add app.txt
git commit -m "docs: adiciona comentário interno"
```
Verifique o histórico
```bash
git log --oneline --graph
```

## 4.3 – Usar git bisect para encontrar o culpado
```bash
# Inicia a busca binária
git bisect start

# Marca o commit atual (com bug visível) como "ruim"
git bisect bad

# Marca o último commit onde sabemos que estava tudo OK
git bisect good HEAD~6

# Roda o teste automaticamente em cada passo
git bisect run ./tests/test.sh

# Encerra o bisect
git bisect reset
```

## 4.4 Como corrigir o bug
1. Faça as alterações necessárias no código para corrigir o bug.
2. Adicione os arquivos corrigidos ao stage:
```bash
git add app.txt
```
3. Crie um commit de fixup apontando para o commit com bug:
```bash
git commit --fixup=<hash-do-commit>
```
4. Reescreva o histórico para juntar a correção ao commit original:
```bash
git rebase -i --autosquash <hash-do-commit>^
```
5. Siga as instruções do rebase (salve e feche o editor).

Se o repositório já foi enviado para o remoto, será necessário forçar o push:
```bash
git push --force
```


















# BLOCO 5 — GitHub Actions CI
1. Crie a pasta e o arquivo de workflow
```bash
mkdir -p .github/workflows
```
```bash
cat > .github/workflows/pr-tests.yml <<'YAML'
name: PR Tests

on:
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run test script
        run: |
          chmod +x tests/test.sh
          ./tests/test.sh
YAML
```
2. Adicione, commit e push
```bash
git add .github/workflows/pr-tests.yml
git commit -m "ci: adiciona workflow que roda test.sh em PRs"
git push
```
> Ao abrir qualquer Pull Request no GitHub, a aba Checks mostrará o job “PR Tests” passando ou falhando conforme o script.

3. Adicione um commit com e abra uma PR para ver a action funcionar
```bash
git checkout -b test/git-actions
echo "Testa o git actions ao abrir uma PR" >> app.txt
git add .
git commit -m "ci: testar git actions em PRs"
git push origin test/git-actions
```















# BLOCO 6 — Hook pre-commit
1. Crie a pasta de hooks se ela não existir
```bash
mkdir -p .git/hooks
```
2. Crie o arquivo .git/hooks/pre-commit
```bash
cat > .git/hooks/pre-commit <<'EOF'

#!/usr/bin/env bash
# Bloqueia commit contendo 'console.log' em arquivos JS
# Ignora linhas que estão sendo removidas (começam com '-').

# Arquivos JS: procura console.log em linhas adicionadas
if git diff --cached --name-only | grep -E '\.js$' >/dev/null; then
    if git diff --cached | grep -E '^\+' | grep -q 'console\.log'; then
        echo "❌ Commit bloqueado: remova console.log antes de commitar."
        exit 1
    fi
fi

exit 0
EOF
```
Verifique se o arquivo foi criado
```bash
nano .git/hooks/pre-commit
```
```bash
chmod +x .git/hooks/pre-commit
```
3. Teste rapidamente
```bash
echo "console.log('teste');" > teste.js
git add teste.js
```
O hook deve bloquear:
```bash
git commit -m "test: tentativa com console.log"
```
```bash
git reset HEAD teste.js && rm teste.js
```

