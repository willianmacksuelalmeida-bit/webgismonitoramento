# LAMSIS-AL — WebGIS de Monitoramento Sismológico de Alagoas

> **Como usar este arquivo:** coloque-o na raiz do repositório do projeto. O Claude Code o lê automaticamente em toda sessão e usará estas instruções como contexto permanente do projeto.

## 1. Visão geral

WebGIS de monitoramento sísmico com foco no estado de Alagoas e abrangência na região Nordeste do Brasil. O mapa exibe epicentros de tremores de terra e as estações sismográficas da rede NB (LabSis/UFRN), sobre malhas de estados e municípios do IBGE.

- **Dados reais já processados e validados** (ver §5): 550 tremores entre 2018 e 2026, em 62 localidades de Alagoas (maiores concentrações: Arapiraca, Craíbas e Belo Monte), com magnitudes de 0,9 a 2,7 nas escalas mR e MLv — 27 eventos com magnitude indeterminada; e 21 estações sismográficas.
- **Público:** pesquisadores, estudantes, defesa civil e público geral interessado na sismicidade regional.
- **Trabalho único do site:** permitir visualizar *onde* e *quando* ocorreram tremores, com que magnitude, e onde estão as estações que os registram — com filtros simples.
- **Referência visual aprovada:** `prototipo-monitoramento-sismico.html` (entregue junto). Reproduzir esse layout, extraindo o CSS/JS para arquivos próprios.

## 2. Stack e decisões técnicas

- **Frontend:** HTML + CSS + JavaScript vanilla com **Leaflet 1.9+**. Sem framework — o escopo não pede React.
- **Sem backend na fase 1:** os GeoJSONs são arquivos estáticos em `data/`, carregados com `fetch()`.
- **Build/dev:** **Vite** (`npm create vite@latest`) ou `python -m http.server`. Confirmar com o autor.
- **Importante:** `fetch()` de GeoJSON **não funciona** abrindo `index.html` via `file://`. Sempre rodar com servidor local.
- **Hospedagem alvo:** GitHub Pages, Netlify ou Vercel (site estático).

## 3. Estrutura de pastas

```
webgis-sismico-al/
├── index.html
├── CLAUDE.md                        ← este arquivo
├── css/style.css
├── js/
│   ├── config.js                    ← mapeamento de campos (§5)
│   ├── layers.js  filters.js  app.js
├── data/                            ← usar o pacote PRONTO (dados-webgis-sismico.zip)
│   ├── epicentros.geojson           ← 550 eventos, 2018–2026 (normalizado e enriquecido)
│   ├── estacoes.geojson             ← 21 estações rede NB
│   ├── municipios_alagoas.geojson   ← 102 municípios (simplificado, 308 KB)
│   └── estados_nordeste.geojson     ← 9 UFs (simplificado, 236 KB)
└── assets/logo.svg
```

⚠️ **Usar os arquivos do pacote `dados-webgis-sismico.zip`**, não os brutos originais — o pacote já resolve merge, datas, join espacial e simplificação (detalhes no §5).

## 4. Camadas

| # | Camada | Fonte | Estilo | Interação |
|---|--------|-------|--------|-----------|
| 1 | Base — Clássico | tiles OpenStreetMap | padrão | rádio no menu |
| 2 | Base — Satélite | tiles Esri World Imagery | padrão | rádio no menu |
| 3 | Base — Relevo | tiles Esri World Shaded Relief (`maxNativeZoom:13`, ampliar tiles além disso) | padrão | rádio no menu |
| 4 | Estados do Nordeste | `data/estados_nordeste.geojson` | contorno preto `#111111` 1.8px, preenchimento 3% | tooltip `NM_UF (SIGLA)`; hover teal |
| 5 | Municípios de Alagoas | `data/municipios_alagoas.geojson` | contorno preto `#111111` 0.8px | tooltip `NM_MUN`; hover teal |
| 6 | Estações sismográficas | `data/estacoes.geojson` | triângulo SVG preto `#0E0E0E` com contorno branco 1.9px (destaca em qualquer mapa base) | popup: código, localização, rede, coordenadas |
| 7 | Epicentros | `data/epicentros.geojson` | círculo, raio `4 + magnitude*2.2` (nula → 5), cor pela rampa (nula → cinza `#6E7F90`) | popup: chip com `mag_txt`, classe, data pt-BR + hora UTC, cidade–UF, coordenadas, fonte; alvo dos filtros |

**Empilhamento (baixo → cima):** base → estados → municípios → estações → epicentros, via `map.createPane()` com zIndex crescente.

**Rampa de magnitude:** micro `< 2,0` `#EFC94C` · menor `2,0–2,9` `#F08A24` · leve `3,0–3,9` `#E14E2A` · moderado `≥ 4,0` `#B3202E`. Eventos M ≥ 4,0 ganham anel pulsante em CSS (nenhum no dataset atual, mas manter para dados futuros; desativar com `prefers-reduced-motion`).

## 5. Contrato de dados — CAMPOS REAIS (já verificados)

### `epicentros.geojson` (processado)
Fonte única: `tremores_2015_2025.geojson` (substitui os 4 arquivos anteriores, restritos a Arapiraca). Campos originais: `Data`, `Dia da Semana`, `Hora - UTC`, `Latitude`, `Longitude`, `Magnitude` (texto, ex.: `"1.5 mR"`) e `Local` (ex.: `"Craíbas AL"`). Properties após processamento:

| Campo | Tipo | Exemplo | Observação |
|-------|------|---------|------------|
| `data` | string ISO | `"2024-05-01"` | dois formatos de origem tratados: `DD/MM/YYYY` e `YYYY.MM.DD` |
| `ano` | int | `2024` | derivado, para o filtro de período |
| `hora` | string \| null | `"07:28"` | de `Hora - UTC` (HH:MM); exibir com sufixo "UTC" |
| `magnitude` | float \| null | `1.5` | número extraído de `Magnitude`; `"<1 mR"` → `0.9`; `"??? mR"` → `null` (27 eventos) |
| `mag_txt` | string | `"1.5 mR"` | texto original — usar no chip do popup |
| `tipo_mag` | string \| null | `"mR"` | escala sismológica: `mR` ou `MLv` |
| `cidade` | string | `"Craíbas"` | `Local` sem o sufixo " AL", grafias normalizadas — alimenta o filtro por cidade (62 valores) |
| `uf` | string | `"AL"` | do rótulo original (sempre AL) |
| `uf_coord` | string \| null | `"AL"` | UF onde a coordenada realmente cai (join espacial) — campo de validação, **não exibir no popup** |
| `fonte` | string | `"LabSis/UFRN"` | fixa |

Sem campo de **profundidade** — não exibir essa linha no popup.

### `estacoes.geojson` (normalizado)
Campos originais → normalizados: `Estação`→`codigo`, `Rede`→`rede`, `Localização`→`localizacao`. Os campos `Lat. (º)`/`Lon. (º)` e `#` foram descartados (a geometria já traz as coordenadas).

### `municipios_alagoas.geojson` / `estados_nordeste.geojson`
Malhas IBGE simplificadas (mapshaper, `-simplify 12%`/`5% keep-shapes` + redução de precisão). Campos mantidos: `CD_MUN`, `NM_MUN` | `CD_UF`, `NM_UF`, `SIGLA`.

### `js/config.js`
```js
export const FIELDS = {
  epicentro: { data:"data", ano:"ano", hora:"hora", mag:"magnitude", magTxt:"mag_txt", cidade:"cidade", uf:"uf", fonte:"fonte" },
  estacao:   { codigo:"codigo", rede:"rede", local:"localizacao" },
  municipio: { nome:"NM_MUN", cod:"CD_MUN" },
  estado:    { nome:"NM_UF", sigla:"SIGLA" }
};
```

### ⚠️ Alertas de qualidade (confirmar com o autor antes de "corrigir")
1. **Código `SBBR` duplicado** nas estações: Sobral/CE e Arquipélago de São Pedro e São Paulo — provável erro de digitação em uma delas.
2. **Duas estações fora do Nordeste continental** (`RBRC` no Acre e a do arquipélago): mantidas; ficam fora do enquadramento inicial.
3. **Arquivo de tremores nomeado `2015_2025`, mas os dados vão de 2018 a 2026** (há um evento em 24/06/2026).
4. **27 eventos `"??? mR"`** (magnitude indeterminada): cinza `#6E7F90`, raio fixo 5; aparecem apenas com o slider de magnitude em 0. **11 eventos `"<1 mR"`** recebem `0.9`.
5. **5 linhas duplicadas exatas** (mesma data/hora/coordenadas/magnitude, em 4 grupos): mantidas no pacote — decidir com o autor se remove.
6. **8 eventos com coordenadas fora de AL** (`uf_coord` = SE/PE/BA) apesar da cidade rotulada AL — ex.: Arapiraca 20/07/2020 → SE; Propiá e Poço Verde são municípios sergipanos rotulados AL. 2 eventos "Plataforma Continental" caem no mar (`uf_coord` nulo).
7. **Precisão das coordenadas × rótulo municipal (verificado — NÃO é erro de SRC):** todas as camadas estão em CRS84/SIRGAS 2000, os pontos coincidem exatamente com os campos `Latitude`/`Longitude` e não há deslocamento sistemático. Porém as coordenadas têm 2 casas decimais (±550 m) — e **64 eventos têm apenas 1 casa** (±5,5 km) —, então **97 dos 550 eventos caem fora do município rotulado** (mediana ~3 km do limite; 48 casos a >3 km; máximo 208 km, provável erro de digitação). O rótulo `Local` do boletim é o município de referência, não necessariamente o polígono da coordenada. Lista completa no anexo `verificacao-epicentros-divergentes.csv` (fora do repositório) — se o autor corrigir coordenadas na origem, reprocessar o pacote.
8. **Grafias mescladas na normalização** do campo `Local`: Olho d'Água das Flores, Quebrangulo, Arapiraca e Traipu tinham variantes de caixa/acento/apóstrofo.

## 6. Funcionalidades

**Fase 1 — MVP** (tudo já demonstrado no protótipo)
1. Vista inicial: `fitBounds` de **Alagoas** `[-10.55,-38.35] → [-8.75,-35.1]` (todos os eventos estão em AL). Botões de zoom rápido "Alagoas" e "Nordeste" (`[-18.6,-48.9] → [-0.8,-34.4]`).
2. Troca de mapa base por rádio; liga/desliga de camadas por checkbox.
3. Popups e tooltips conforme §4, campos conforme §5.
4. Legenda fixa no menu; escala métrica; zoom no canto superior direito.

**Fase 2 — Filtros e resumo**
1. Filtro por **cidade**: select "Todas as cidades" + 62 opções em ordem alfabética pt-BR, derivadas dos dados.
2. Magnitude mínima (slider 0–4, passo 0,1). Regra para indeterminadas: com o slider em 0, todos os eventos aparecem; acima de 0, eventos com `magnitude = null` são ocultados.
3. Período por ano inicial/final (derivar `anoMin`/`anoMax` dos dados; hoje 2018–2026).
4. Painel-resumo (4 KPIs): eventos visíveis, maior magnitude (ignorando nulas), "cidades com registro sísmico" e período com anos completos (`2018–2026`). Os três filtros combinam entre si.

**Fase 3 — Implementado no protótipo**
1. **Gráfico de eventos por ano** em SVG puro (sem dependência), clicável.
2. **Enquadramento**: botões Alagoas e Nordeste (`flyToBounds`). O botão "Filtrados" foi implementado e **removido a pedido do autor**.

> A exportação de dados (CSV/GeoJSON) chegou a ser implementada e foi **removida a pedido do autor** — o site é apenas de consulta. Não reintroduzir sem pedido explícito.

**Fase 4 — Desejável (validar antes)**
- Estado dos filtros na URL (permite compartilhar recortes); export PNG do mapa; busca digitável no select de cidades; camada de calor (heatmap) para densidade.
- Automatizar a incorporação de novos boletins do LabSis (script Python reaproveitando a lógica de merge/join do §5).

## 7. Design

Referência visual: `prototipo-monitoramento-sismico.html`. O padrão adotado é **Data-Dense Dashboard / Real-Time Monitoring** em tema escuro (derivado da skill `ui-ux-pro-max`), com densidade alta e hierarquia clara.

**Layout.** Painel lateral esquerdo de 340px (310px ≤1180px; gaveta deslizante ≤900px com scrim, botão de fechar e tecla Esc). Ordem das seções: Marca → **Mapa base + Enquadramento** → Resumo (KPIs + gráfico) → Filtros → Camadas → Legenda → Desenvolvimento e fontes (seção fixa, sempre visível — não usar `<details>`).

**Marca.** Cabeçalho fixo (`position:sticky`), centralizado: eyebrow "Monitoramento Sísmico"; sigla **LAMSIS - AL** em Chakra Petch 28px, toda em `--text` (sem cor de destaque no sufixo); nome completo "Laboratório de Monitoramento Sismológico de Alagoas" logo abaixo em 11,5px peso 500 na mesma cor do título (`--text`, não `--muted`), obrigatoriamente **em uma única linha** (`white-space:nowrap`; 10,5px ≤420px). O tamanho foi calibrado medindo a largura real do texto em Archivo — 12px estoura o painel de 340px; e sismograma com pulso de luz âmbar contínuo (dasharray `70 570`, 3,2s linear infinito).

**Tokens.** Superfícies `#0B121A` / `#101B26` / `#16222F` / `#1B2938`; linhas `#243447` / `#2E4257`; texto `#E9EEF4` / `#9DB0C2` / `#74889C`; acento teal `#3BC7CB` (`#1F6E71` para estado ativo). Magnitude: `#EFC94C` · `#F08A24` · `#E14E2A` · `#D93646` · indeterminada `#7C8D9E`. Tipografia: Chakra Petch (títulos), Archivo (interface), IBM Plex Mono (números). Transições 180ms.

**Acessibilidade (verificada).** Todas as combinações de texto ≥4,5:1 e elementos gráficos ≥3:1. Os valores originais `#B3202E` (2,43:1) e `#6E7F90` (3,91:1) **falhavam** e foram substituídos por `#D93646` e `#7C8D9E`. O chip de magnitude troca para texto branco quando M ≥ 4,0. Demais requisitos: `:focus-visible` teal em todos os controles, ícones em SVG (nunca emoji), `aria-pressed` nos segmented controls, `aria-live="polite"` nos KPIs, `role="status"` no aviso de resultado vazio, `role="alert"` no erro de carregamento, `prefers-reduced-motion` desliga todas as animações, e nenhuma informação transmitida só por cor (magnitude tem cor + tamanho + rótulo textual).

**Componentes próprios.** KPIs em grade 2×2 com conteúdo centralizado (flex column, `text-align:center`); minigráfico de barras SVG por ano (clicável — define o período; barras fora do período ficam esmaecidas); segmented controls para mapa base e enquadramento; linhas de camada com contagem; swatch de linha para camadas de limite e de cor para pontos; botão "Limpar" que só aparece com filtro ativo; leitura de coordenadas + zoom no canto do mapa; overlay de carregamento com spinner; aviso flutuante quando o filtro não retorna eventos.

## 8. Critérios de aceite

- As 3 bases alternam corretamente e as 4 camadas temáticas carregam de `data/` sem erro no console; nenhum popup exibe `undefined`.
- Filtros de cidade, magnitude e período funcionam em conjunto e atualizam os KPIs e o gráfico (550 eventos / 62 cidades / máx. 2,7 com filtros zerados).
- Contraste ≥4,5:1 (texto) e ≥3:1 (gráficos); navegação completa por teclado com foco visível; `prefers-reduced-motion` respeitado.
- Testado em 1440px, 768px e 390px: painel vira gaveta ≤900px, com botão de fechar, scrim e Esc.
- Tooltips mostram os 102 municípios e as 9 UFs pelo nome.
- Funciona em Chrome e Firefox, desktop e mobile (menu recolhível).
- Publicado (GitHub Pages ou equivalente); payload de dados ≈ 550 KB já garantido pela simplificação.

## 9. Roteiro de execução

1. Criar estrutura e setup (Vite ou servidor simples) — confirmar com o autor.
2. Extrair `dados-webgis-sismico.zip` para `data/`.
3. Portar o protótipo para a estrutura multi-arquivo (`css/`, `js/` com `config.js` do §5), trocando os dados embutidos por `fetch()` de `data/`.
4. Implementar/refinar Fase 2 e revisar critérios do §8.
5. Revisão com o autor → ajustes → deploy.
