# Projeto

Primeiro é preciso entender que as funções de ***Changelog*** e de publicação da versão estão juntas no mesmo workflow, que está no arquivo `.github/workflows/changelog.yml`.

## Explicação (linha a linha)

### 1. Quando o workflow de publicação é iniciado

```yaml
on:
  push: 
    tags:
      - v[0-9]+.[0-9]+.[0-9]+
```

- Esse trecho define quando o workflow será iniciado: quando uma tag for enviada para o GitHub.
- A tag é um nome que identifica uma versão do projeto. O padrão configurado indica versões como `v1.1.0`.

Exemplo de envio de uma tag:

```bash
git push origin v1.1.0
```

### 2. Job: Deploy

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```

- O ***Job*** `deploy` reúne as etapas necessárias para publicar a versão.
- `runs-on: ubuntu-latest` indica que essas etapas serão executadas em um ambiente Linux fornecido pelo GitHub.

### 3. Permissão para atualizar o projeto

```yaml
permissions:
  contents: write
```

- Essa permissão permite que o workflow crie a publicação da versão e salve a atualização do `CHANGELOG.md` no repositório.

### 4. Etapas do Job

O Job `deploy` possui alguns ***Steps*** (etapas), executados na ordem em que aparecem no arquivo.

#### 4.1. Baixar os arquivos do projeto

```yaml
- name: Checkout Code
  uses: actions/checkout@v5
```

- Essa etapa disponibiliza os arquivos do repositório no ambiente onde o workflow está sendo executado.
- Assim, as etapas seguintes conseguem consultar e atualizar arquivos do projeto.

#### 4.2. Preparar as informações do Changelog

```yaml
- name: Update CHANGELOG
  id: changelog
  uses: requarks/changelog-action@v1
  with:
    token: ${{ github.token }}
    tag: ${{ github.ref_name }}
```

- Essa etapa usa uma Action pronta para buscar as mudanças relacionadas à versão e preparar o texto do changelog.
- `github.ref_name` representa o nome da tag que iniciou o workflow.
- O resultado dessa etapa recebe o identificador `changelog`, para poder ser usado na etapa seguinte.

#### 4.3. Criar a publicação da versão

```yaml
- name: Create Release
  uses: ncipollo/release-action@v1.20.0
  with:
    allowUpdates: true
    draft: false
    makeLatest: true
    name: ${{ github.ref_name }}
    body: ${{ steps.changelog.outputs.changes }}
    token: ${{ github.token }}
```

- Essa etapa cria uma publicação de versão (*Release*) no GitHub usando o nome da tag.
- O texto preparado na etapa anterior é usado como descrição das mudanças dessa versão.
- `draft: false` indica que a publicação não será criada como rascunho.
- `makeLatest: true` indica que essa publicação deve aparecer como a versão mais recente.
- `allowUpdates: true` permite atualizar uma publicação com o mesmo nome caso ela já exista.

#### 4.4. Salvar o Changelog no repositório

```yaml
- name: Commit CHANGELOG.md
  uses: stefanzweifel/git-auto-commit-action@v7
  with:
    branch: main
    commit_message: 'docs: update CHANGELOG.md for ${{ github.ref_name }} [skip ci]'
    file_pattern: CHANGELOG.md
```

- Essa etapa salva a atualização do `CHANGELOG.md` na branch `main`.
- A mensagem do commit identifica a versão que foi publicada.
- `[skip ci]` informa que esse commit automático não precisa iniciar novamente o workflow de CI.
- `file_pattern: CHANGELOG.md` limita esse commit às alterações feitas nesse arquivo.

### 5. Resumo do fluxo de publicação

Quando uma tag de versão é enviada, o workflow baixa os arquivos, prepara o texto das mudanças, cria a publicação no GitHub e salva o `CHANGELOG.md` atualizado na branch `main`.

## Como executar a Action de Changelog

Depois que o workflow estiver na branch padrão do repositório, acompanhe as execuções pela aba **Actions**.

### 1. Conferir se as Actions estão habilitadas

1. Abra o repositório no GitHub.
2. Acesse **Settings > Actions > General**.
3. Em **Actions permissions**, permita o uso de Actions conforme a configuração da organização ou do repositório.
4. Se houver a opção **Workflow permissions**, selecione permissões de leitura e escrita para que o workflow de publicação possa criar a Release e atualizar o `CHANGELOG.md`. O workflow também declara `contents: write`.
5. Salve as alterações, se necessário.

### 2. Publicar uma versão e atualizar o Changelog

O workflow `Deploy`, definido em `.github/workflows/changelog.yml`, é iniciado automaticamente quando uma tag de versão é enviada. Ele não possui o gatilho manual `workflow_dispatch`.

1. Confira se as alterações que devem ser publicadas já estão na branch `main`.
2. No terminal, atualize as referências e crie uma tag seguindo o formato `vMAJOR.MINOR.PATCH`, por exemplo:

   ```bash
   git checkout main
   git pull origin main
   git tag v1.3.5
   ```

3. Envie a tag para o GitHub:

   ```bash
   git push origin v1.3.5
   ```

4. No GitHub, abra **Actions** e acompanhe a execução de **Deploy** iniciada pela tag.
5. Quando todas as etapas terminarem com sucesso, confira a nova Release em **Releases** e a atualização do `CHANGELOG.md` na branch `main`.

Cada versão deve usar uma tag nova. Para publicar outra versão, incremente o número apropriado e repita os passos.

### 3. Se uma execução não iniciar ou falhar

- Confirme que o workflow está na branch padrão do repositório e que Actions estão habilitadas.
- Para o Deploy, confirme que enviou uma tag no formato `v1.2.3` com `git push origin <tag>`. Apenas criar a tag localmente não inicia o workflow.
- Na aba **Actions**, abra a execução com falha e consulte a etapa marcada em vermelho para ver os detalhes do erro.
- Se a etapa que salva o `CHANGELOG.md` não tiver permissão, verifique as permissões do workflow em **Settings > Actions > General**.