import 'package:flutter/material.dart';
//------------------------------------------------------------------------------
// 1.Inicial do APP-------------------------------------------------------------
//------------------------------------------------------------------------------
  void main(){runApp(const MyApp());}
  class MyApp extends StatelessWidget{
    const MyApp({super.key});
    @override
    Widget build(BuildContext context) {
      return MaterialApp(
        title: 'Finanças',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),useMaterial3: true,),
        home: const HomeScreen(),);}}
//------------------------------------------------------------------------------
// 2.Organização dos selects----------------------------------------------------
//------------------------------------------------------------------------------
  enum Tipo{Receita,Despesa,Poupar, Resgatar;
    String get label {
      switch (this){
        case Tipo.Receita:  return 'Receita';
        case Tipo.Despesa:  return 'Despesa';
        case Tipo.Poupar:   return 'Poupança';
        case Tipo.Resgatar: return 'Resgate Poupança';}}}

  class Transacao{
    final String id;        final DateTime data;
    final Tipo tipo;        final String categoria;   //'Alimentação', 'Moradia', 'Salario', etc. Cadastrada pelo usuario
    final String descricao; final double valor;
    Transacao({
      required this.id,         required this.data,
      required this.tipo,       required this.categoria,
      required this.descricao,  required this.valor,});}

//------------------------------------------------------------------------------
// 3.Inicio do app--------------------------------------------------------------
//------------------------------------------------------------------------------
  class HomeScreen extends StatefulWidget{const HomeScreen({super.key});
    @override
    State<HomeScreen> createState() => _HomeScreenState();}
  class _HomeScreenState extends State<HomeScreen> {
    final List<String> _categoriasDisponiveis = [
      'Alimentação',
      'Salario',
      'Banco',
      'Transferencia',
    ];
    String? _categoriaSelecionada;
//------------------------------------------------------------------------------
// 3.1.Lista falsa--------------------------------------------------------------
//------------------------------------------------------------------------------
    final List<Transacao> _transacoesFake = [
      Transacao(
        id: '1',
        descricao: 'Salario Principal',
        valor: 100.00,
        tipo: Tipo.Receita,
        categoria: 'Salario',
        data: DateTime.now(),
      ),
      Transacao(
        id: '2',
        descricao: 'Supermercado Mensal',
        valor: 10.00,
        tipo: Tipo.Despesa,
        categoria: 'Alimentacao',
        data: DateTime.now(),
      ),
      Transacao(
          id: '3',
          descricao: 'Poupanca do Mês',
          valor: 20.00,
          tipo: Tipo.Poupar,
          categoria: 'Banco',
          data: DateTime.now(),
      ),
      Transacao(
        id: '4',
        descricao: 'Resgate de Conta Poupança',
        valor: 5.00,
        tipo: Tipo.Resgatar,
        categoria: 'Banco',
        data: DateTime.now(),
      ),
    ];

//------------------------------------------------------------------------------
// 4.Calculos dos saldos--------------------------------------------------------
//------------------------------------------------------------------------------
    double get _saldoTotal{
      double total = 0.0;
      for(var transacao in _transacoesFake){
        if (transacao.tipo == Tipo.Receita || transacao.tipo == Tipo.Resgatar){
          total +=transacao.valor;
        }else if (transacao.tipo == Tipo.Despesa || transacao.tipo == Tipo.Poupar){
          total -=transacao.valor;}}
      return total;}
    double get _saldoPoupanca{
      double total = 0.0;
      for(var transacao in _transacoesFake){
        if (transacao.tipo == Tipo.Poupar){
          total +=transacao.valor;
        }else if (transacao.tipo == Tipo.Resgatar){
          total -=transacao.valor;}}
      return total;}
//------------------------------------------------------------------------------
// 5.Formulario de Cadastro de Categorias---------------------------------------
//------------------------------------------------------------------------------
    void _abrirFormularioCategoria(BuildContext context) {
      final TextEditingController _novaCategoriaController = TextEditingController();

      showModalBottomSheet(
        context: context,
        isScrollControlled: true, // Garante que o teclado não cubra o campo de texto
        shape: const RoundedRectangleBorder(
          borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
        ),
        builder: (ctx) {
          return Padding(
            padding: EdgeInsets.only(
              bottom: MediaQuery.of(ctx).viewInsets.bottom + 20, // Empurra o modal para cima do teclado
              top: 20,
              left: 16,
              right: 16,
            ),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                Text(
                  'Nova Categoria',
                  style: Theme.of(context).textTheme.titleLarge?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 15),
                TextField(
                  controller: _novaCategoriaController,
                  decoration: const InputDecoration(
                    labelText: 'Nome da Categoria',
                    hintText: 'Ex: Combustível, Mercado, Farmácia',
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.category),
                  ),
                  textCapitalization: TextCapitalization.sentences, // Primeira letra maiúscula automática
                  autofocus: true,
                ),
                const SizedBox(height: 20),
                ElevatedButton(
                  onPressed: () {
                    final text = _novaCategoriaController.text.trim();

                    if (text.isEmpty) return;

                    // Evita cadastrar categorias duplicadas
                    if (_categoriasDisponiveis.any((c) => c.toLowerCase() == text.toLowerCase())) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(content: Text('Esta categoria já existe!')),
                      );
                      return;
                    }

                    // Atualiza o estado adicionando a nova categoria à lista
                    setState(() {
                      _categoriasDisponiveis.add(text);
                    });

                    Navigator.pop(ctx); // Fecha o formulário de categoria

                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(
                        content: Text('Categoria "$text" adicionada com sucesso!'),
                        backgroundColor: Colors.teal,
                      ),
                    );
                  },
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.teal,
                    foregroundColor: Colors.white,
                    padding: const EdgeInsets.symmetric(vertical: 12),
                  ),
                  child: const Text('Salvar Categoria', style: TextStyle(fontSize: 16)),
                ),
              ],
            ),
          );
        },
      );
    }
//------------------------------------------------------------------------------
// 5.Formulario de Movimentação-------------------------------------------------
//------------------------------------------------------------------------------
    final _dataController = TextEditingController();
    final _descricaoController = TextEditingController();
    final _valorController = TextEditingController();
    Tipo _tipoSelecionado = Tipo.Despesa;
    String _CategoriaSelecionada = 'Salario';
    void _abrirFormularioMovimentacao(BuildContext context){
      _dataController.clear();
      _descricaoController.clear();
      _valorController.clear();
      _tipoSelecionado = Tipo.Despesa;
      _CategoriaSelecionada = 'Salario';
      showModalBottomSheet(
        context: context,
        isScrollControlled: true,
        builder: (ctx){
          return Padding(
            padding: EdgeInsets.only(
              top: 24,left: 24,right: 24,bottom: MediaQuery.of(ctx).viewInsets.bottom + 24,
            ),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                const Text(
//Novo Lançamento
                  'Novo Lançamento', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                  textAlign: TextAlign.center,),
                const SizedBox(height: 16),
//Data
                TextField(
                  controller: _dataController,
                  decoration: const InputDecoration(labelText: 'dd/mm/yyyy'),
                ),
                const SizedBox(height: 12),
//Tipo
                Row(
                  children: [
                    Expanded(
                      child: DropdownButtonFormField<Tipo>(
                        initialValue: _tipoSelecionado,
                        decoration: const InputDecoration(labelText: 'Tipo'),
                        items: const [
                          DropdownMenuItem(value: Tipo.Despesa, child: Text('Despesa')),
                          DropdownMenuItem(value: Tipo.Receita, child: Text('Receita')),
                          DropdownMenuItem(value: Tipo.Poupar, child: Text('Poupar')),
                          DropdownMenuItem(value: Tipo.Resgatar, child: Text('Resgatar')),
                        ],
                        onChanged: (val) => _tipoSelecionado = val!,
                      ),
                    ),
                    const SizedBox(width: 16),
//Categoria
                    Expanded(
                      child: DropdownButtonFormField<String>(
                        initialValue: _CategoriaSelecionada,
                        decoration: const InputDecoration(labelText: 'Categoria'),
                        items: const [
                          DropdownMenuItem(value: 'Salario', child: Text('Salario')),
                          DropdownMenuItem(value: 'Banco', child: Text('Banco')),
                          DropdownMenuItem(value: 'Transferencia', child: Text('Transferencia')),
                        ],
                        onChanged: (val) => _CategoriaSelecionada = val!,
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 16),
//Descrição
                TextField(
                  controller: _descricaoController,
                  decoration: const InputDecoration(labelText: 'Descrição (EX: Conta de Luz'),
                ),
                const SizedBox(height: 12),
//Valor
                TextField(
                  controller: _valorController,
                  keyboardType: const TextInputType.numberWithOptions(decimal: true),
                  decoration: const InputDecoration(labelText: 'Valor (R\$)'),
                ),
                const SizedBox(height: 24),
//Butão
                ElevatedButton(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.green,
                    foregroundColor: Colors.white,
                    padding: const EdgeInsets.symmetric(vertical: 14),
                  ),
                  onPressed:(){
                    final desc = _descricaoController.text;
                    final valor = double.tryParse(_valorController.text) ?? 0.0;
                    if  (desc.isEmpty || valor <=0) {
                      return;
                    }
                    setState(() {
                      _transacoesFake.add(
                        Transacao(
                          id: DateTime.now().toString(),
                          descricao: desc,
                          valor: valor,
                          tipo: _tipoSelecionado,
                          categoria: _CategoriaSelecionada,
                          data: DateTime.now(),
                        ),
                      );
                    });
                    Navigator.of(context).pop();
                  },
                  child: const Text('Confirmar Lançamento', style: TextStyle(fontSize: 16)),
                ),
              ],
            ),
          );
        },
      );
    }

    @override
    Widget build(BuildContext context){
      return Scaffold(
        appBar: AppBar(
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
//-----------------------------------------------------------------------------
//Titulo do app
//-----------------------------------------------------------------------------
          title: const Text('Finanças da Familia Silva'),
          centerTitle: true,
        ),
        body: Column(
          children: [
            //bloco do topo:Exibição do Saldo = cartão de banco
            Card(
              margin: const EdgeInsets.all(16.0),
              elevation: 4,
              child: Padding(
                padding: const EdgeInsets.all(24.0),
                child: Column(
                  children: [
                    // 1. Linha do Saldo Geral
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text(
//-----------------------------------------------------------------------------
//Saldo Atual
//-----------------------------------------------------------------------------
                          'Saldo Atual: ', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                        ),
                        Text(
                          'R\$ ${_saldoTotal.toStringAsFixed(2)}',
                          style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold, color: Colors.green[700]),
                        ),
                      ],
                    ),

                    const Divider(height: 24, thickness: 1),

                    // 2. Linha da Poupança
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Row(
                          children: [
                            Icon(Icons.savings, size: 20, color: Colors.blue),
                            SizedBox(width: 8),
                            Text(
//-----------------------------------------------------------------------------
//Saldo Poupança
//-----------------------------------------------------------------------------
                              'Poupança:', style: TextStyle(fontSize: 16, color: Colors.black54),
                            ),
                          ],
                        ),
                        Text(
                          'R\$ ${_saldoPoupanca.toStringAsFixed(2)}',
                          style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.blue[700]),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),

            const Padding(
              padding: EdgeInsets.symmetric(horizontal: 16.0),
              child: Align(
                alignment: Alignment.centerLeft,
                child: Text(
//-----------------------------------------------------------------------------
//Tela dos ultimos lançamentos
//-----------------------------------------------------------------------------
                  'Ultimo Lançamento',
                  style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                ),
              ),
            ),

            //Lista de transações(Mesmo que caregar tabela do DB na Tela
            Expanded(
              child: ListView.builder(
                itemCount: _transacoesFake.length,
                itemBuilder: (context, index){
                  final transacao = _transacoesFake[index];

                  //Define a cor do icone com base no tipo
                  Color corIcone = Colors.grey;
                  IconData icone = Icons.compare_arrows;

                  if(transacao.tipo == Tipo.Receita){
                    corIcone = Colors.green;
                    icone = Icons.arrow_upward;
                  }else if(transacao.tipo == Tipo.Despesa){
                    corIcone = Colors.red;
                    icone = Icons.arrow_downward;
                  }else if(transacao.tipo == Tipo.Poupar){
                    corIcone = Colors.blue;
                    icone = Icons.move_to_inbox;
                  }else if(transacao.tipo == Tipo.Resgatar){
                    corIcone = Colors.green;
                    icone = Icons.unarchive;
                  }

                  return ListTile(
                    leading: CircleAvatar(
                      backgroundColor: corIcone.withAlpha(30), //Fundo levemente colorido
                      child:Icon(icone, color: corIcone),
                    ),
                    title: Text('${transacao.data.day} / ${transacao.data.month} --> ${transacao.descricao}'),
                    trailing: Text(
                      'R\$ ${transacao.valor.toStringAsFixed(2)}',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        //Definir a cor do valor dinamicamente
                        color: transacao.tipo == Tipo.Receita || transacao.tipo == Tipo.Resgatar
                          ? Colors.green
                          :(transacao.tipo == Tipo.Despesa  || transacao.tipo == Tipo.Poupar
                            ? Colors.red
                            : Colors.grey),
                      ),
                    ),
                  );

                },
              )
            )
          ],
        ),

        //Botão Flutuante O classico + no canto Inferior
        floatingActionButton: PopupMenuButton<String>(
          child: Material(
            elevation: 6,
            shape: const CircleBorder(),
            color: Theme.of(context).colorScheme.primaryContainer,
            child: const Padding(
              padding: EdgeInsets.all(16.0),
              child: Icon(Icons.add, size: 28),
            ),
          ),
          offset: const Offset(0, -120),
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(15)),
          onSelected: (valor){
            if (valor == 'Movimentacao'){
              _abrirFormularioMovimentacao(context);
            }else if (valor == 'Categoria'){
              _abrirFormularioCategoria(context);
            }
          },
          itemBuilder: (BuildContext context) => <PopupMenuEntry<String>>[
            const PopupMenuItem<String>(
              value: 'Movimentacao',
              child: Row(
                children:[
                  Icon(Icons.swap_horiz, color: Colors.blue),
                  SizedBox(width: 10),
                  Text('Movimentação'),
                ],
              ),
            ),
            const PopupMenuDivider(),
            const PopupMenuItem<String>(
              value:'Categoria',
              child: Row(
                children:[
                  Icon(Icons.category, color: Colors.teal),
                  SizedBox(width: 10),
                  Text('Categoria'),
                ],
              ),
            ),
          ],
        ),
      );
    }
  }
