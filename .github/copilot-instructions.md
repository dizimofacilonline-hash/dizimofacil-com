# Instruções para Copilot - Projeto Dízimo Fácil

## Visão Geral do Projeto

**Dízimo Fácil** é um sistema web para gerenciamento de dizimistas em paróquias. Compreende dois domínios principais:

- **www.dizimofacil.com** (`dizimofacil/`): Landing page institucional com apresentação do serviço
- **Sistema de Gestão** (`douglasmiiller.com/`): Interface web para consulta de dizimistas com banco de dados MySQL

## Arquitetura e Componentes

### 1. Landing Page (`dizimofacil/index.html`)

- **Propósito**: Apresentação do produto e canal de contato
- **Stack**: HTML5 + CSS3 puro (sem frameworks)
- **Design**: Responsive com gradiente roxo (`#667eea` → `#764ba2` → `#f093fb`)
- **Animações**: Fade-in e slide-down CSS nativos
- **Integração externa**: Link WhatsApp direto (`https://wa.me/5527996068115`)
- **Padrões**:
  - Usar `Inter` como fonte padrão (importada do Google Fonts)
  - Classes de animação reutilizáveis (`fadeIn`, `slideDown`, `slideUp`)
  - Media queries para mobile-first responsividade (320px, 400px, 640px)

### 2. Sistema de Gestão (`douglasmiiller.com/`)

#### Fluxo de Autenticação
```
login.php → PDO MySQL → SESSION['logged'] → painel.php (não público)
```

**Credenciais**: Comparação hash em `login.php` (usuário/senha não expostos em GET)

#### Consulta de Dizimistas

**Dois modos de busca** (complementares):

1. **ConsultarDizimista.php**: Busca por código (ID) do dizimista
   - Campo input numérico, validação regex `[^0-9]`
   - Conecta ao banco `DIZIMISTAS`, coluna `ID`
   - Retorna: nome, nascimento, celular, fixo, situação, observação

2. **ConsultarDizimistaNome.php**: Busca por nome
   - Implementa normalização de acentos (`removerAcentos()`)
   - Permite busca fuzzy sem preocupação com diacríticos
   - Retorna lista de resultados com seleção via link `?codigo=X`

#### Validação de Campos Incompletos

Para dizimistas com `SITUACAO = 'ATIVO'`:
- **Nascimento**: Vazio, `0000-00-00`, `--/--/----`, ou contém `--`
- **Celular**: Vazio, `(00)00000-0000`, `SEM CELULAR`, ou contém `(00)`

Campos incompletos aparecem em destaque amarelo com alerta visual.

### 3. Banco de Dados

**Tabela DIZIMISTAS** (MySQL):
```sql
ID (INT, PK) | NOMEDIZIMISTA | DATANASCIMENTO | CELULAR | FIXO | SITUACAO | DATACADASTRO | OBSERVACAO
```

**Valores esperados para SITUACAO**: `ATIVO`, `INATIVO`, `FALECIDO`

**Conexão**:
- Host: `mysql.douglasmiiller.com`
- PDO + charset `utf8mb4`
- Tratamento de exceção: `PDO::ERRMODE_EXCEPTION`

## Padrões e Convenções

### PHP
- **OOP**: Uso de PDO (prepared statements, não raw SQL)
- **Sessions**: Validação via `$_SESSION['logged']`
- **HTML escape**: Sempre usar `htmlspecialchars()` para output
- **Configuração**: Credentials em variáveis globais (revise para `.env` em produção)

### CSS
- **Colors**: Paleta gradiente consistente (roxo → magenta)
- **Spacing**: Margin/padding em múltiplos de 0.25rem ou 15px
- **Sombras**: `box-shadow: 0 20px 60px rgba(0,0,0,0.15)` padrão para cards
- **Transições**: `all 0.3s ease` padrão
- **Borderradius**: 8-24px dependendo do contexto (buttons: 50px, cards: 16-24px)

### UI/UX
- **Estados visuais**: Badges por situação (ATIVO verde, INATIVO vermelho, FALECIDO cinza)
- **Feedback**: Mensagens de erro em fundo rosa, alertas em amarelo
- **Hover effects**: Elevação com `translateY(-2px)` + aumento de shadow
- **Acessibilidade**: Placeholders, labels para inputs, alt text em imagens

## Fluxos Críticos

### Adicionar Nova Página de Consulta
1. Criar `ConsultarDizimista[Criterio].php` com função `connectDB()`
2. Implementar SELECT prepare/bind pattern (PDO)
3. Espelhar validação de campos incompletos se ATIVO
4. Usar template de resultado com classes de situação (`.resultado-box.inativo` etc)
5. Incluir link alternativo para "Outra opção de busca"

### Atualizar Validação de Campos
- Editar função de verificação em `ConsultarDizimista.php` linhas ~60-85
- Sincronizar com `ConsultarDizimistaNome.php` (mesmo bloco lógico)
- Testar com dados de situações diferentes (ATIVO/INATIVO/FALECIDO)

### Deploy
- Certifique-se de que `config.php` (se existir) **não é versionado** (`.gitignore`)
- Credenciais MySQL devem vir via ambiente, não hardcoded
- HTTPS obrigatório em produção

## Pontos de Extensão Futuros

- **Admin Panel** (`/sgd/painel.php`): Gerenciamento completo de dizimistas
- **Autenticação**: Considerar JWT ou cookies seguros ao invés de sessions básicas
- **Sanitização**: Input validation além de HTML escape (libphonenumber, date validation)
- **Performance**: Índices MySQL em `NOMEDIZIMISTA` para buscas de nome
