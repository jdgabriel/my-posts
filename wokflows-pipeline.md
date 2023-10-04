# Criando pipelines eficientes com Github Actions

## Introdução

Com a evolução do código e da complexidade do repositório, é muito comum nas organizações a necessidade de criar pipelines para automatizar os fluxos das tarefas necessárias para manter a integração contínua e entrega contínua (CI/CD).
Para que não precise escrever e manter a mesma pipeline múltiplas vezes em vários repositórios, uma alternativa é distribuir seu pipeline como um produto. Mantendo centralizadas as pipelines necessárias, para consulta e execução.

## O que é Github Actions?

GitHub Actions é uma plataforma de integração contínua e entrega contínua (CI/CD) que permite automatizar suas tarefas de compilações, testes e qualquer função ou fluxo de trabalho necessário para sua aplicação, em produção ou desenvolvimento. Para entender melhor como funciona, consulte a [documentação oficial](https://docs.github.com/pt/actions/learn-github-actions/understanding-github-actions).

## Básico de uma Pipeline

Para adicionar uma pipeline action ao seu repositório, na raiz, crie uma pasta **.github/workflows**. Esta pasta será responsável por manter todos os pipelines disponíveis para a Github Actions.

Por exemplo:

```yml
name: CI
on:
  pull_request:

jobs:
  node:
    runs-on: ubuntu-20.04
    steps:
      - name: Baixar meu código
        uses: actions/checkout@v2

      - name: Instalar dependencias
        run: npm install

      - name: Realizar testes
        run: npm test
```

Consulte a [documentação](https://docs.github.com/pt/actions/using-workflows/workflow-syntax-for-github-actions) para saber mais sobre as propriedades da action.

## Pipeline reutilizáveis

O que suas pipelines tem em comum?
Normalmente uma tarefa de testes automatizados, uma tarefa de linter, uma tarefa para gerar a imagem Docker.
Já pensou em criar apenas uma pipeline para cada tarefa e reutilizá-las em todos os seu projetos, definindo padrões de entrega? Com Github Actions isso é possível.

### Então vamos começar...

Para organizar as pipelines, crie um repositório para mantê-las.
Crie sua pipeline na pasta **.github/workflows**.

Exemplo de uma pipeline para criar uma imagem Docker, ao fazer um push na branch main:

```yml
name: "Docker Build and Push"

on:
  push:
    branches:
      - main

jobs:
  pipeline-docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Docker Credentials
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Docker Build
        run: docker build -t docker_image -f Dockerfile .

      - name: "Docker Push Image"
        run: |
          docker tag docker_image docker.io/org/docker_image:latest
          docker push -a docker.io/org/docker_image
```

Com o código acima, você terá uma pipeline para Docker. (**.github/worflows/docker-pipeline.yml**)

Para disponibilizar essa pipeline é prciso alterar o trigger para `workflow_call` (Action so é acionada quando chamada, por outra action) e adicionar `inputs` e `secrets` (caso precice).

Vamos adicionar dois `inputs`:

- `os_version` : Para identificar o sistema operacional da pipeline
- `image_name` : Para identificar o nome da imagem gerada pelo Docker

Agora, adicione duas `secrets`:

- `DOCKERHUB_USERNAME` : Usuário do Dockerhub
- `DOCKERHUB_TOKEN` : Token de acesso ao Dockerhub

Então ficaria assim:

```yml
name: "Docker build and Push"

on:
  workflow_call:
    inputs:
      os_version:
        description: "OS Version"
        default: "ubuntu-latest"
        required: false
        type: string
      image_name:
        description: "Docker image name"
        required: true
        type: string
    secrets:
      DOCKERHUB_USERNAME:
        required: true
      DOCKERHUB_TOKEN:
        required: true

jobs:
  pipeline-docker:
    runs-on: ${{ inputs.os_version }}
    steps:
      - uses: actions/checkout@v2

      - name: Docker Credentials
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Docker Build
        run: docker build -t ${{ inputs.image_name }} -f Dockerfile .

      - name: "Docker Push Image"
        run: |
          docker tag docker_image docker.io/org/${{ inputs.image_name }}:latest
          docker push -a docker.io/org/${{ inputs.image_name }}
```

Explicando:

`on.workflow_call.inputs.<input_id>.description`
Uma descrição breve do input.

`on.workflow_call.inputs.<input_id>.default`
Valor definido como padrão, caso a propriedade não seja passada.

`on.workflow_call.inputs.<input_id>.required`
Define se a propriedade é obrigatória, caso defina uma valor `default`, esta pode ser `false`.<br />

`on.workflow_call.inputs.<input_id>.type`
Recebem os valores de `boolean`, `number` e `string`.

`on.workflow_call.secrets.<secret_id>`
Uma string para identificar uma variável.

`on.workflow_call.secrets.<secret_id>.required`
Valor boleano para definir a obrigatoriedade da propridade.

## Ok, como faço para utilizar?

Para distribuir suas pipelines, antes é preciso adicionar uma configuração ao seu **Repositório de Pipelines**.

Vá em:
`Settings > Actions > General > Access`

Marque a opção e salve:
`Accessible from repositories in the 'sua-organization' organization`

![Access Pipeline](https://github.com/jdgabriel/my-posts/blob/main/images/access_workflow.png?raw=true)

Agora com o devido acesso garantido, podemos utilizar a pipeline recem criada.
Para isso crie uma pipeline no repositório do seu código. (**.github/workflows**)

Vamos utilizar a pipeline abaixo como exemplo

```yml
name: CI
on:
  pull_request:

jobs:
  node:
    runs-on: ubuntu-20.04
    steps:
      - name: Baixar meu código
        uses: actions/checkout@v2

      - name: Instalar dependencias
        run: npm install

      - name: Realizar testes
        run: npm test
```

Adicione a pipeline reutilizável que você criou.

```yml
docker:
  uses: sua-org/seu-repo/.github/workflows/docker-pipeline.yml@main
  with:
    os_version: "ubuntu-latest"
    image_name: "minha-imagem-docker"
  secrets:
    DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
    DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
    DOCKERHUB_ORGANIZATION: ${{ secrets.DOCKERHUB_ORGANIZATION }}
```

Na propriedade `uses` coloque a url da sua pipeline com a referência (**@ref**), que pode ser o nome da branch, uma tag ou uma hash do commit. Defina todos os `inputs` da pipeline na propriedade `with` e os `secrets` necessários na propriedade `secrets`. Agora sua pipeline está completa, basta executar.

## Conclusão

As Pipelines reutilizáveis são uma ótima alternativa para centralizar e organizar todos os fluxos necessários para uma integração contínua e entrega contínua (CI/CD) da su organização.
Mas comece simples, construa pipelines que façam coisas simples e vá aumentando a complexidade sempre que possível. Pequenas pipelines como uma de linter, por exemplo, vão fazer a diferença na experiência do desenvolvedor e do time como um todo.
