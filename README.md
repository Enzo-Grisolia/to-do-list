# To-Do List Android — FIAP

Aplicativo Android desenvolvido individualmente para a atividade de **Android Development da FIAP**. O objetivo é disponibilizar um fluxo completo de gerenciamento de tarefas com persistência local, permitindo **listar, cadastrar, editar, concluir/desmarcar e excluir tarefas**, além de navegar entre a listagem e o formulário.

## Funcionalidades implementadas

- Listagem de tarefas em `LazyColumn`.
- Cadastro de novas tarefas.
- Edição de tarefas existentes.
- Marcação e desmarcação de tarefa como concluída.
- Exclusão de tarefas.
- Persistência local com Room.
- Navegação entre lista e formulário com Navigation Compose.
- Estado da interface controlado por ViewModel e `StateFlow`.
- Previews das principais telas Compose.

## Tecnologias utilizadas

- Kotlin
- Jetpack Compose
- Material 3
- Room
- Coroutines e Flow
- ViewModel
- Navigation Compose
- KSP
- Gradle

## Arquitetura

O projeto separa persistência, acesso aos dados, estado da aplicação e interface em responsabilidades diferentes.

Fluxo principal:

```text
Compose UI
   ↓
TarefaViewModel
   ↓
TarefaRepository
   ↓
TarefaDao
   ↓
Room / SQLite
```

### TarefaRepository

`TarefaRepository` funciona como intermediário entre a `TarefaViewModel` e o `TarefaDao`. Ele expõe o `Flow<List<Tarefa>>` fornecido pelo DAO e concentra as operações de inserção, atualização e exclusão. Dessa forma, a camada de apresentação não precisa acessar o banco diretamente.

### TarefaViewModel

`TarefaViewModel` mantém os dados necessários para a interface e executa as operações da aplicação. A lista fornecida pelo Repository é transformada em um `StateFlow<List<Tarefa>>` usando `stateIn` e `viewModelScope`.

As operações de inserir, atualizar e excluir são executadas em coroutines através de `viewModelScope.launch`. A própria ViewModel também disponibiliza uma `Factory`, responsável por obter o `TarefaDao` do `TarefaDatabase`, criar o Repository e então criar a ViewModel.

### ListaTarefasScreen

`ListaTarefasScreen` observa `viewModel.tarefas` com `collectAsStateWithLifecycle()`. Quando o `StateFlow` recebe uma nova lista, o Compose atualiza a interface automaticamente.

A tela apresenta as tarefas em uma `LazyColumn` e dispara ações para a ViewModel:

- checkbox: atualiza o campo `concluida`;
- toque no card: abre a edição da tarefa;
- botão de exclusão: remove a tarefa;
- botão flutuante: inicia o cadastro de uma nova tarefa.

Também existem previews para visualizar a listagem com tarefas e a lista vazia.

### FormularioTarefaScreen

O mesmo formulário atende cadastro e edição. A rota informa um `tarefaId`:

- `tarefaId = 0`: modo de cadastro;
- `tarefaId != 0`: modo de edição.

No modo de edição, a tarefa correspondente é localizada no estado da ViewModel e seus dados são utilizados para preencher título e descrição. Ao salvar, o formulário insere uma nova `Tarefa` ou atualiza a tarefa existente e retorna à tela anterior.

O botão Salvar só fica habilitado quando o título possui conteúdo. A tela possui previews para os modos de nova tarefa e edição.

### AppNavigation

`AppNavigation` cria o `NavController` e configura duas rotas principais:

```text
lista
formulario/{tarefaId}
```

A aplicação inicia em `lista`. Para cadastrar uma tarefa, a navegação abre `formulario/0`. Para editar, o ID da tarefa é enviado na rota, por exemplo `formulario/3`.

Ao finalizar ou tocar no botão de voltar, `popBackStack()` retorna à listagem sem encerrar o aplicativo.

### MainActivity

A `MainActivity` deixou de utilizar o conteúdo de exemplo criado pelo template do Android Studio. Dentro de `setContent`, ela aplica o tema, cria a `TarefaViewModel` usando `TarefaViewModel.factory(applicationContext)` e inicia `AppNavigation`.

Isso conecta o fluxo completo:

```text
MainActivity → AppNavigation → Screens → ViewModel → Repository → Room
```

## Persistência local

A entidade `Tarefa` é armazenada na tabela `tarefas` e possui:

- `id`: chave primária gerada automaticamente;
- `titulo`;
- `descricao`;
- `concluida`;
- `dataCriacao`.

`TarefaDao` fornece consulta da lista por ordem de criação e operações de `Insert`, `Update` e `Delete`. `TarefaDatabase` cria o banco local `tarefas.db` usando Room.

## Como executar

### Pré-requisitos

- Android Studio compatível com o projeto;
- JDK 11;
- Android SDK configurado;
- emulador Android ou dispositivo físico com Android 7.0 / API 24 ou superior.

### Android Studio

1. Clone este repositório.
2. Abra a pasta do projeto no Android Studio.
3. Aguarde a sincronização do Gradle.
4. Selecione um emulador ou dispositivo físico.
5. Execute o módulo `app`.

### Linha de comando

No Windows:

```bash
./gradlew.bat assembleDebug
```

No Linux/macOS:

```bash
./gradlew assembleDebug
```

O APK de debug é gerado em `app/build/outputs/apk/debug/`.

## Estrutura principal

```text
app/src/main/java/com/github/enzo_grisolia/to_do_list/
├── MainActivity.kt
├── data/
│   ├── Tarefa.kt
│   ├── TarefaDao.kt
│   └── TarefaDatabase.kt
├── navigation/
│   └── AppNavigation.kt
├── repository/
│   └── TarefaRepository.kt
├── ui/
│   ├── FormularioTarefaScreen.kt
│   ├── ListaTarefasScreen.kt
│   └── theme/
└── viewmodel/
    └── TarefaViewModel.kt
```

## Evidências

As evidências de execução devem ser armazenadas em [`docs/evidencias`](docs/evidencias), conforme solicitado na atividade. A pasta contém a relação de capturas necessárias e os nomes recomendados para os arquivos.

Evidências previstas:

1. tela inicial com a lista de tarefas;
2. cadastro de uma nova tarefa;
3. tarefa cadastrada aparecendo na lista;
4. edição de uma tarefa existente;
5. tarefa marcada como concluída;
6. exclusão de uma tarefa;
7. navegação entre lista e formulário;
8. build ou execução sem erros.

> As imagens de execução devem ser capturadas em um emulador ou dispositivo real. Elas não são geradas artificialmente neste repositório.

## Autor

**Enzo Grisolia** — atividade individual FIAP.
