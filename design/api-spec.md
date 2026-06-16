# API Specification — Catálogo de Filmes e Séries

Base URL: `http://localhost:5000`  
Content-Type: `application/json`  
Documentação interativa: `http://localhost:5000/swagger`

---

## Enums de referência

| Enum | Valores |
|---|---|
| `Genero` | `Acao`, `Comedia`, `Drama`, `Terror`, `Romance`, `SciFi`, `Documentario`, `Animacao`, `Thriller`, `Aventura`, `Fantasia`, `Crime` |
| `StatusSerie` | `EmAndamento`, `Finalizada`, `Cancelada`, `Pausada` |
| `ClassificacaoEtaria` | `Livre`, `Dez`, `Doze`, `Quatorze`, `Dezesseis`, `Dezoito` |

---

## Schemas compartilhados

### `ErroResponse`
```json
{
  "erro": "string",
  "detalhes": ["string"]
}
```

### `ObraResumoResponse`
```json
{
  "id": "guid",
  "titulo": "string",
  "genero": "Genero",
  "anoLancamento": 0,
  "avaliacao": 0.0,
  "tipo": "Filme | Serie"
}
```

---

## /filmes

### GET `/filmes`

Lista todos os filmes cadastrados.

**Query params**

| Param | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `genero` | `Genero` | Não | Filtra por gênero |
| `classificacao` | `ClassificacaoEtaria` | Não | Filtra por classificação etária |

**Response `200`**
```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "titulo": "Matrix",
    "genero": "SciFi",
    "anoLancamento": 1999,
    "sinopse": "Um hacker descobre a verdade sobre sua realidade.",
    "avaliacao": 8.7,
    "duracaoMinutos": 136,
    "diretor": "Lana Wachowski",
    "classificacao": "Quatorze",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
]
```

---

### GET `/filmes/{id}`

Retorna um filme pelo ID.

**Path params**

| Param | Tipo | Descrição |
|---|---|---|
| `id` | `guid` | ID do filme |

**Response `200`** — `FilmeResponse` (mesmo schema acima)  
**Response `404`** — `ErroResponse`

---

### POST `/filmes`

Cadastra um novo filme.

**Request body — `CriarFilmeRequest`**
```json
{
  "titulo": "string",           // obrigatório, max 200
  "genero": "Genero",           // obrigatório
  "anoLancamento": 1999,        // obrigatório, 1888–ano atual
  "sinopse": "string",          // opcional, max 2000
  "avaliacao": 8.7,             // obrigatório, 0.0–10.0
  "duracaoMinutos": 120,        // obrigatório, > 0
  "diretor": "string",          // obrigatório, max 150
  "classificacao": "Livre"      // obrigatório
}
```

**Response `201`** — `FilmeResponse` + header `Location: /filmes/{id}`  
**Response `400`** — `ErroResponse` (validação de campos)  
**Response `422`** — `ErroResponse` (violação de regra de domínio)

---

### PUT `/filmes/{id}`

Substitui todos os dados de um filme.

**Request body — `AtualizarFilmeRequest`** — mesmos campos de `CriarFilmeRequest`

**Response `200`** — `FilmeResponse`  
**Response `400`** — `ErroResponse`  
**Response `404`** — `ErroResponse`

---

### DELETE `/filmes/{id}`

Remove um filme do catálogo. Remove também de todos os favoritos de usuários.

**Response `204`** — sem corpo  
**Response `404`** — `ErroResponse`

---

## /series

### GET `/series`

Lista todas as séries.

**Query params**

| Param | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `genero` | `Genero` | Não | Filtra por gênero |
| `status` | `StatusSerie` | Não | Filtra por status |

**Response `200`**
```json
[
  {
    "id": "guid",
    "titulo": "Breaking Bad",
    "genero": "Drama",
    "anoLancamento": 2008,
    "sinopse": "Um professor de química se torna um fabricante de drogas.",
    "avaliacao": 9.5,
    "numeroTemporadas": 5,
    "numeroEpisodiosPorTemporada": 13,
    "criador": "Vince Gilligan",
    "status": "Finalizada",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
]
```

---

### GET `/series/{id}`

**Response `200`** — `SerieResponse`  
**Response `404`** — `ErroResponse`

---

### POST `/series`

**Request body — `CriarSerieRequest`**
```json
{
  "titulo": "string",                       // obrigatório, max 200
  "genero": "Genero",                       // obrigatório
  "anoLancamento": 2008,                    // obrigatório, 1888–ano atual
  "sinopse": "string",                      // opcional, max 2000
  "avaliacao": 9.5,                         // obrigatório, 0.0–10.0
  "numeroTemporadas": 5,                    // obrigatório, >= 1
  "numeroEpisodiosPorTemporada": 13,        // obrigatório, >= 1
  "criador": "string",                      // obrigatório, max 150
  "status": "EmAndamento"                   // obrigatório
}
```

**Response `201`** — `SerieResponse` + header `Location: /series/{id}`  
**Response `400`** — `ErroResponse`  
**Response `422`** — `ErroResponse`

---

### PUT `/series/{id}`

**Request body — `AtualizarSerieRequest`** — mesmos campos de `CriarSerieRequest`

**Response `200`** — `SerieResponse`  
**Response `400`** / `404`** — `ErroResponse`

---

### DELETE `/series/{id}`

**Response `204`** — sem corpo  
**Response `404`** — `ErroResponse`

---

## /catalogo

### GET `/catalogo`

Lista todas as obras (filmes + séries combinados), com filtragem opcional.

**Query params**

| Param | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `tipo` | `filme \| serie` | Não | Filtra por tipo de obra |
| `genero` | `Genero` | Não | Filtra por gênero |

**Response `200`** — `ObraResumoResponse[]`

---

### GET `/catalogo/buscar`

Pesquisa obras pelo título (case-insensitive, busca parcial).

**Query params**

| Param | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `titulo` | `string` | Sim | Texto a buscar no título |

**Response `200`** — `ObraResumoResponse[]`  
**Response `400`** — `ErroResponse` (quando `titulo` está vazio)

---

### GET `/catalogo/genero/{genero}`

Lista todas as obras de um gênero específico.

**Path params**

| Param | Tipo | Descrição |
|---|---|---|
| `genero` | `Genero` | Gênero a filtrar |

**Response `200`** — `ObraResumoResponse[]`  
**Response `400`** — `ErroResponse` (gênero inválido)

---

## /usuarios

### GET `/usuarios`

**Response `200`**
```json
[
  {
    "id": "guid",
    "nome": "string",
    "email": "string",
    "totalFavoritos": 3,
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

---

### GET `/usuarios/{id}`

**Response `200`** — `UsuarioResponse`  
**Response `404`** — `ErroResponse`

---

### POST `/usuarios`

**Request body — `CriarUsuarioRequest`**
```json
{
  "nome": "string",     // obrigatório, max 100
  "email": "string"     // obrigatório, formato e-mail válido, único no sistema
}
```

**Response `201`** — `UsuarioResponse` + header `Location: /usuarios/{id}`  
**Response `400`** — `ErroResponse`  
**Response `409`** — `ErroResponse` (e-mail já cadastrado)

---

### PUT `/usuarios/{id}`

**Request body — `AtualizarUsuarioRequest`**
```json
{
  "nome": "string",     // obrigatório, max 100
  "email": "string"     // obrigatório, formato e-mail válido
}
```

**Response `200`** — `UsuarioResponse`  
**Response `400`** / `404`** — `ErroResponse`  
**Response `409`** — `ErroResponse` (e-mail já pertence a outro usuário)

---

### DELETE `/usuarios/{id}`

**Response `204`** — sem corpo  
**Response `404`** — `ErroResponse`

---

## /usuarios/{id}/favoritos

### GET `/usuarios/{id}/favoritos`

Retorna todas as obras favoritadas pelo usuário, com detalhes completos.

**Response `200`** — `ObraResumoResponse[]`  
**Response `404`** — `ErroResponse` (usuário não encontrado)

---

### POST `/usuarios/{id}/favoritos/{obraId}`

Adiciona uma obra aos favoritos do usuário.

**Response `204`** — sem corpo  
**Response `404`** — `ErroResponse` (usuário ou obra não encontrada)  
**Response `409`** — `ErroResponse` (obra já está nos favoritos)

---

### DELETE `/usuarios/{id}/favoritos/{obraId}`

Remove uma obra dos favoritos do usuário.

**Response `204`** — sem corpo  
**Response `404`** — `ErroResponse` (usuário não encontrado ou obra não está nos favoritos)

---

## Swagger — anotações obrigatórias

Cada endpoint deve ter as seguintes anotações XML para o Swagger:

```csharp
/// <summary>Descrição curta do endpoint</summary>
/// <param name="id">Descrição do parâmetro</param>
/// <response code="200">Descrição do retorno de sucesso</response>
/// <response code="404">Recurso não encontrado</response>
[ProducesResponseType(typeof(FilmeResponse), StatusCodes.Status200OK)]
[ProducesResponseType(typeof(ErroResponse), StatusCodes.Status404NotFound)]
```

Todo controller deve ter `[Produces("application/json")]` e `[Consumes("application/json")]`.

O `Program.cs` deve incluir:
```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Catálogo de Filmes e Séries",
        Version = "v1",
        Description = "API REST para gerenciamento de catálogo audiovisual"
    });
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});
```
