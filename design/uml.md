# UML — Catálogo de Filmes e Séries

---

## 1. Diagrama de Classes — Domínio Completo

```mermaid
classDiagram
    direction TB

    class ObraAudiovisual {
        <<abstract>>
        +Guid Id
        +string Titulo
        +Genero Genero
        +int AnoLancamento
        +string Sinopse
        +decimal Avaliacao
        +DateTime CreatedAt
        +DateTime UpdatedAt
        +ObterDetalhes() string*
    }

    class Filme {
        +int DuracaoMinutos
        +string Diretor
        +ClassificacaoEtaria Classificacao
        +ObterDetalhes() string
    }

    class Serie {
        +int NumeroTemporadas
        +int NumeroEpisodiosPorTemporada
        +string Criador
        +StatusSerie Status
        +ObterDetalhes() string
    }

    class Usuario {
        +Guid Id
        +string Nome
        +string Email
        +IReadOnlyList~Guid~ Favoritos
        +DateTime CreatedAt
        +AdicionarFavorito(Guid obraId) void
        +RemoverFavorito(Guid obraId) void
        +PossuiFavorito(Guid obraId) bool
    }

    class Catalogo {
        -List~ObraAudiovisual~ obras
        +Adicionar(ObraAudiovisual obra) void
        +Remover(Guid id) void
        +ObterPorId(Guid id) ObraAudiovisual
        +ListarTodos() IReadOnlyList~ObraAudiovisual~
        +BuscarPorTitulo(string titulo) IReadOnlyList~ObraAudiovisual~
        +FiltrarPorGenero(Genero genero) IReadOnlyList~ObraAudiovisual~
    }

    class Genero {
        <<enumeration>>
        Acao
        Comedia
        Drama
        Terror
        Romance
        SciFi
        Documentario
        Animacao
        Thriller
        Aventura
        Fantasia
        Crime
    }

    class StatusSerie {
        <<enumeration>>
        EmAndamento
        Finalizada
        Cancelada
        Pausada
    }

    class ClassificacaoEtaria {
        <<enumeration>>
        Livre
        Dez
        Doze
        Quatorze
        Dezesseis
        Dezoito
    }

    ObraAudiovisual <|-- Filme : herda
    ObraAudiovisual <|-- Serie : herda

    ObraAudiovisual --> Genero : usa
    Filme           --> ClassificacaoEtaria : usa
    Serie           --> StatusSerie : usa

    Catalogo "1" o-- "0..*" ObraAudiovisual : agrega
    Usuario  "1" --> "0..*" ObraAudiovisual : favorita (por Id)
```

---

## 2. Diagrama de Relacionamentos

```mermaid
erDiagram
    USUARIO {
        Guid    id
        string  nome
        string  email
        DateTime createdAt
    }

    FAVORITO {
        Guid usuarioId
        Guid obraId
    }

    OBRA_AUDIOVISUAL {
        Guid    id
        string  titulo
        string  genero
        int     anoLancamento
        string  sinopse
        decimal avaliacao
        string  tipo
        DateTime createdAt
        DateTime updatedAt
    }

    FILME {
        int    duracaoMinutos
        string diretor
        string classificacao
    }

    SERIE {
        int    numeroTemporadas
        int    episodiosPorTemporada
        string criador
        string status
    }

    USUARIO      ||--o{ FAVORITO         : "possui"
    FAVORITO     }o--||  OBRA_AUDIOVISUAL : "referencia"
    OBRA_AUDIOVISUAL ||--o| FILME         : "é um"
    OBRA_AUDIOVISUAL ||--o| SERIE         : "é uma"
```

---

## 3. Arquitetura em Camadas

```mermaid
graph TD
    HTTP["Client HTTP"]

    subgraph api["Camada API"]
        C["Controllers"]
        D["DTOs Request / Response"]
        M["GlobalExceptionMiddleware"]
    end

    subgraph app["Camada Application"]
        SI["IFilmeService\nISerieService\nIUsuarioService\nICatalogoService"]
        S["FilmeService\nSerieService\nUsuarioService\nCatalogoService"]
    end

    subgraph domain["Camada Domain"]
        E["Entities\nObraAudiovisual · Filme · Serie\nUsuario · Catalogo"]
        EN["Enums\nGenero · StatusSerie · ClassificacaoEtaria"]
        EX["Exceptions\nDomainException · NotFoundException"]
    end

    subgraph infra["Camada Infrastructure"]
        RI["IRepository&lt;T&gt;"]
        R["FilmeRepository\nSerieRepository\nUsuarioRepository\n(in-memory)"]
    end

    HTTP --> C
    C    --> SI
    S    -.->|implementa| SI
    S    --> RI
    R    -.->|implementa| RI
    S    --> E
    R    --> E

    style domain fill:#f5f0ff,stroke:#9b59b6
    style app    fill:#eaf4fb,stroke:#2980b9
    style api    fill:#eafaf1,stroke:#27ae60
    style infra  fill:#fef9e7,stroke:#f39c12
```

---

## 4. Fluxo — Adicionar Favorito

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant FavoritosController
    participant UsuarioService
    participant CatalogoService
    participant UsuarioRepository

    Client            ->> FavoritosController : POST /usuarios/{id}/favoritos/{obraId}
    FavoritosController ->> UsuarioService     : AdicionarFavorito(usuarioId, obraId)
    UsuarioService    ->> CatalogoService      : ObterPorId(obraId)
    CatalogoService  -->> UsuarioService       : ObraAudiovisual  [ou NotFoundException]
    UsuarioService    ->> UsuarioRepository    : GetById(usuarioId)
    UsuarioRepository -->> UsuarioService      : Usuario  [ou NotFoundException]
    UsuarioService    ->> UsuarioService       : usuario.AdicionarFavorito(obraId)
    UsuarioService    ->> UsuarioRepository    : Update(usuario)
    UsuarioService   -->> FavoritosController  : void
    FavoritosController -->> Client            : 204 No Content
```

---

## 5. Fluxo — Busca no Catálogo

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant CatalogoController
    participant CatalogoService
    participant Catalogo

    Client               ->> CatalogoController : GET /catalogo/buscar?titulo=matrix
    CatalogoController   ->> CatalogoService    : BuscarPorTitulo("matrix")
    CatalogoService      ->> Catalogo           : BuscarPorTitulo("matrix")
    Catalogo            -->> CatalogoService    : IReadOnlyList~ObraAudiovisual~
    CatalogoService     -->> CatalogoController : IReadOnlyList~ObraResumoResponse~
    CatalogoController  -->> Client             : 200 [ ObraResumoResponse, ... ]
```
