# javaInterfaceConcept

Perfeito! Você quer uma **janela principal única com um menu lateral** (como uma barra de navegação) e, ao clicar em um botão nessa barra, o painel da direita muda para o **componente correspondente** (ex: listagem de livros, formulário de cadastro, etc.).

Essa abordagem é bastante comum em interfaces modernas e funciona bem com **Swing**.

---

## 🧩 Estrutura Geral da Janela

* 🧭 **Menu lateral (esquerda)**: `JPanel` com botões como:

  * 📖 "Listar Livros"
  * ➕ "Adicionar Livro"
* 🪟 **Área principal (direita)**: `JPanel` que muda de conteúdo com `CardLayout`

---

## 🧱 Esboço Final do Código

### 1. `JanelaPrincipal.java`

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.util.List;
import controller.AcervoManager;

public class JanelaPrincipal extends JFrame {

    private JPanel painelConteudo;      // Painel à direita com CardLayout
    private CardLayout cardLayout;      // Gerenciador de layout
    private AcervoManager manager;

    public JanelaPrincipal() {
        super("Sistema de Livros");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(900, 600);
        setLocationRelativeTo(null);
        setLayout(new BorderLayout());

        manager = new AcervoManager();

        // Menu lateral
        JPanel menuLateral = new JPanel();
        menuLateral.setLayout(new BoxLayout(menuLateral, BoxLayout.Y_AXIS));
        menuLateral.setPreferredSize(new Dimension(200, 0));

        JButton btnListar = new JButton("📚 Listar Livros");
        JButton btnCadastrar = new JButton("➕ Adicionar Livro");

        menuLateral.add(btnListar);
        menuLateral.add(btnCadastrar);

        add(menuLateral, BorderLayout.WEST);

        // Painel principal com CardLayout
        cardLayout = new CardLayout();
        painelConteudo = new JPanel(cardLayout);

        // Telas a serem alternadas
        TelaListagem telaListagem = new TelaListagem(manager);
        TelaCadastro telaCadastro = new TelaCadastro(manager, telaListagem);

        painelConteudo.add(telaListagem, "listar");
        painelConteudo.add(telaCadastro, "cadastrar");

        add(painelConteudo, BorderLayout.CENTER);

        // Ações dos botões
        btnListar.addActionListener((ActionEvent e) -> cardLayout.show(painelConteudo, "listar"));
        btnCadastrar.addActionListener((ActionEvent e) -> cardLayout.show(painelConteudo, "cadastrar"));

        setVisible(true);
    }
}
```

---

### 2. `TelaListagem.java` – Tabela com busca dinâmica

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import java.awt.*;
import java.io.IOException;
import java.util.List;
import java.util.stream.Collectors;
import controller.AcervoManager;
import model.Acervo;

public class TelaListagem extends JPanel {
    private JTable tabela;
    private JTextField campoBusca;
    private AcervoTableModel modelo;
    private List<Acervo> todos;

    public TelaListagem(AcervoManager manager) {
        setLayout(new BorderLayout());

        campoBusca = new JTextField();
        add(campoBusca, BorderLayout.NORTH);

        try {
            todos = manager.carregar();
        } catch (IOException e) {
            todos = List.of();
        }

        modelo = new AcervoTableModel(todos);
        tabela = new JTable(modelo);
        add(new JScrollPane(tabela), BorderLayout.CENTER);

        campoBusca.getDocument().addDocumentListener(new DocumentListener() {
            public void insertUpdate(DocumentEvent e) { filtrar(); }
            public void removeUpdate(DocumentEvent e) { filtrar(); }
            public void changedUpdate(DocumentEvent e) { filtrar(); }

            private void filtrar() {
                String termo = campoBusca.getText().toLowerCase();
                List<Acervo> filtrado = todos.stream()
                        .filter(ac -> ac.getLivro().getTitulo().toLowerCase().contains(termo))
                        .collect(Collectors.toList());
                modelo.setAcervos(filtrado);
            }
        });
    }

    public void atualizarDados(List<Acervo> novaLista) {
        this.todos = novaLista;
        modelo.setAcervos(novaLista);
    }
}
```

---

### 3. `TelaCadastro.java` – Formulário para adicionar livros

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import controller.AcervoManager;
import model.Livro;
import model.Acervo;
import java.io.IOException;
import java.util.List;

public class TelaCadastro extends JPanel {
    private JTextField txtTitulo, txtAutor, txtEditora, txtIdioma, txtIsbn, txtQtd;
    private AcervoManager manager;
    private TelaListagem telaListagem;

    public TelaCadastro(AcervoManager manager, TelaListagem telaListagem) {
        this.manager = manager;
        this.telaListagem = telaListagem;

        setLayout(new GridLayout(7, 2, 5, 5));

        txtTitulo = new JTextField();
        txtAutor = new JTextField();
        txtEditora = new JTextField();
        txtIdioma = new JTextField();
        txtIsbn = new JTextField();
        txtQtd = new JTextField();

        add(new JLabel("Título:")); add(txtTitulo);
        add(new JLabel("Autor:")); add(txtAutor);
        add(new JLabel("Editora:")); add(txtEditora);
        add(new JLabel("Idioma:")); add(txtIdioma);
        add(new JLabel("ISBN:")); add(txtIsbn);
        add(new JLabel("Quantidade:")); add(txtQtd);

        JButton btnSalvar = new JButton("Salvar");
        add(btnSalvar);

        btnSalvar.addActionListener((ActionEvent e) -> {
            try {
                Livro livro = new Livro(
                        txtTitulo.getText(),
                        txtAutor.getText(),
                        txtEditora.getText(),
                        txtIdioma.getText(),
                        txtIsbn.getText()
                );
                int qtd = Integer.parseInt(txtQtd.getText());
                Acervo ac = new Acervo(livro, qtd);

                List<Acervo> lista = manager.carregar();
                lista.add(ac);
                manager.salvar(lista);
                telaListagem.atualizarDados(lista);

                JOptionPane.showMessageDialog(this, "Livro salvo!");
                limparCampos();
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(this, "Erro: " + ex.getMessage());
            }
        });
    }

    private void limparCampos() {
        txtTitulo.setText("");
        txtAutor.setText("");
        txtEditora.setText("");
        txtIdioma.setText("");
        txtIsbn.setText("");
        txtQtd.setText("");
    }
}
```

---

### 4. `Main.java`

```java
public class Main {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(JanelaPrincipal::new);
    }
}
```

---

### 🧪 Resultado

Você terá uma janela como esta:

```
+------------------------+----------------------------------+
| [📚 Listar Livros]     | [Tabela de livros filtrável]     |
| [➕ Adicionar Livro]   | [Ou formulário de cadastro]      |
+------------------------+----------------------------------+
```

Ao clicar em cada botão no menu lateral, o painel direito muda de acordo.

---

Quer que eu te envie um `.zip` com esse projeto pronto para usar no IntelliJ ou Eclipse?
