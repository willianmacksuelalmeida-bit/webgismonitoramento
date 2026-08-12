# LAMES — WebGIS de Monitoramento Sísmico de Alagoas

Site estático pronto para publicação. Não precisa de servidor de aplicação nem banco de dados.

```
index.html                        ← o mapa
data/epicentros.geojson           ← 550 eventos, 2018–2026
data/estacoes.geojson             ← 21 estações (rede NB)
data/municipios_alagoas.geojson   ← 102 municípios
data/estados_nordeste.geojson     ← 9 UFs
CLAUDE.md                         ← especificação técnica do projeto
```

## Recursos da interface

- Filtros combinados por cidade, magnitude mínima e período, com gráfico de eventos por ano clicável.
- Painel de indicadores: eventos visíveis, maior magnitude, cidades atingidas e período.
- Exportação dos eventos filtrados em CSV e GeoJSON.
- Três mapas base (clássico, satélite, relevo) e enquadramento rápido (Alagoas, Nordeste, filtrados).
- Tema escuro com contraste verificado (WCAG AA), navegação por teclado e layout responsivo.

## Publicar no GitHub Pages

1. Crie um repositório público (ex.: `webgis-lames`).
2. Envie **o conteúdo desta pasta** para a raiz do repositório (o `index.html` precisa ficar na raiz, não dentro de outra pasta).
3. No repositório: **Settings → Pages → Source: Deploy from a branch → Branch: `main` / `/ (root)` → Save**.
4. Em 1–2 minutos o site fica disponível em `https://SEU-USUARIO.github.io/webgis-lames/`.

Os dados ficam acessíveis por URL própria e podem ser consumidos por outros sistemas (QGIS, Leaflet, ArcGIS):

```
https://SEU-USUARIO.github.io/webgis-lames/data/epicentros.geojson
```

## Alternativas

- **Netlify / Vercel:** arraste a pasta na interface — publica em segundos, com HTTPS e domínio próprio opcional.
- **Servidor institucional (UNEAL/NUPEA):** copie a pasta para o diretório público do servidor web (`public_html`, `www` ou equivalente).

## Testar localmente antes

```bash
python -m http.server 8000
# abra http://localhost:8000
```

⚠️ Abrir o `index.html` com duplo clique (`file://`) **não funciona**: o navegador bloqueia o carregamento dos GeoJSONs. Sempre use um servidor.

## Atualizar os dados

Substitua o arquivo correspondente em `data/` mantendo o mesmo nome e a mesma estrutura de campos (ver `CLAUDE.md`, §5). O mapa lê os dados a cada carregamento — não é preciso mexer no `index.html`.

---

Dados sísmicos e estações: LabSis/UFRN (rede NB). Malhas territoriais: IBGE.
Desenvolvimento: Willian Macksuel Almeida Melo — NUPEA/UNEAL.
