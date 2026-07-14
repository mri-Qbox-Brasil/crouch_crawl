# crouch_crawl — Manual

Recurso standalone que adiciona agachar e rastejar (de bruços ou de costas), preservando o walkstyle do jogador.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Comandos](#comandos)
5. [Controles](#controles)
6. [Comportamento](#comportamento)
7. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
8. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| — | — | Standalone. O `fxmanifest.lua` não declara nenhuma dependência |

---

## Instalação

1. Copie a pasta `crouch_crawl` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure crouch_crawl
   ```
3. Não há SQL nem itens de inventário.
4. **Conflitos** — o RPEmotes já embute este mesmo sistema. Se você usa RPEmotes com `Config.CrouchEnabled` ou `Config.CrawlEnabled` ativos, desative um dos dois lados para não registrar dois keybinds e duas rotinas de animação sobre o mesmo ped.

---

## Configuração

Arquivo: `config.lua`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Config.CrouchKeybindEnabled` | bool | Sim | Registra o keymapping de agachar. Se `false`, só resta o comando `/crouch_mri_Q` |
| `Config.CrouchKeybind` | string | Sim | Tecla padrão de agachar. Padrão: `LCONTROL` |
| `Config.CrouchOverrideStealthMode` | bool | Sim | Só tem efeito quando a tecla de agachar é a mesma do `INPUT_DUCK` (Left Control). Se `true`, o modo furtivo é ignorado e o jogador agacha direto. Se `false`, o jogador só agacha se já estiver em modo furtivo |
| `Config.CrouchKeypressTimer` | número (ms) | Sim | Janela máxima entre dois toques na tecla para entrar em agachado quando `CrouchOverrideStealthMode = false`. Padrão: `1000` |
| `Config.CrawlKeybindEnabled` | bool | Sim | Registra o keymapping de rastejar. Se `false`, só resta o comando `/crawl` |
| `Config.CrawlKeybind` | string | Sim | Tecla padrão de rastejar. Padrão: `RCONTROL` |
| `Config.Localization` | tabela | Sim | Textos exibidos. Chaves: `crouch_keymapping`, `crouch_chat_suggestion`, `crawl_keymapping`, `crawl_chat_suggestion` |

---

## Comandos

| Comando | Permissão | Descrição |
|---|---|---|
| `/crouch_mri_Q` | Todos | Alterna agachado. Sempre registrado, mesmo com `CrouchKeybindEnabled = false` |
| `/crawl` | Todos | Alterna rastejar. Sempre registrado, mesmo com `CrawlKeybindEnabled = false` |

Os keymappings usam os comandos internos `+crouch_mri_Q` e `+crawl`, que não devem ser chamados pelo chat.

---

## Controles

| Tecla | Ação |
|---|---|
| `Left Control` (padrão) | Agachar / levantar |
| `Right Control` (padrão) | Deitar (rastejar) / levantar |
| `W` / `S` | Rastejar para frente / para trás |
| `A` / `D` | Girar o corpo enquanto deitado |
| `Espaço` | Virar de bruços para de costas e vice-versa (não é keymapped, usa `INPUT_JUMP`) |

O jogador pode remapear agachar e rastejar em Pausa > Configurações > Atalhos de teclado > FiveM.

---

## Comportamento

- **Walkstyle preservado** — ao agachar, o clipset atual do jogador é lido pelo mapa em `walkstyles.lua` (mais de 150 clipsets do jogo base) e restaurado ao levantar. Se o clipset não estiver no mapa, o recurso volta ao movimento padrão.
- **Sem levantar sozinho** — as ambient anims ficam desligadas enquanto agachado (`SetPedCanPlayAmbientAnims(false)` em loop).
- **Tiro agachado** — permitido, com velocidade de movimento limitada a `0.15` de blend ratio enquanto mirando.
- **Primeira pessoa** — desabilitada enquanto agachado.
- **Mergulho** — se o jogador estiver correndo ao acionar o rastejar, ele executa a animação de mergulho ao chão em vez da de deitar.
- **Troca de arma bloqueada** — enquanto deitado (`CPED_CONFIG_FLAG_BlockWeaponSwitching`).
- **Interrupções** — o estado é cancelado automaticamente se o jogador entra em veículo, cai, fica ferido, entra em ragdoll, entra em combate corpo a corpo ou (no caso do rastejar) entra na água.

---

## Entrypoints para outros recursos

Todos os exports são do lado cliente.

```lua
-- Retorna true se o jogador está agachado
local crouched = exports.crouch_crawl:IsPlayerCrouched()

-- Retorna true se o jogador está deitado (parado ou se movendo)
local prone = exports.crouch_crawl:IsPlayerProne()

-- Retorna true apenas enquanto o jogador se move rastejando (frente/trás)
local crawling = exports.crouch_crawl:IsPlayerCrawling()

-- Retorna "onfront" ou "onback". Sempre retorna string, mesmo se o jogador
-- não estiver deitado — combine com IsPlayerProne() para validar.
local proneType = exports.crouch_crawl:GetPlayerProneType()
```

---

## Estrutura de arquivos

```
crouch_crawl/
├── client.lua        — lógica de agachar/rastejar, keymappings, comandos e exports
├── config.lua        — keybinds, override do modo furtivo e textos
├── walkstyles.lua    — mapa hash → nome de clipset, usado para restaurar o walkstyle
└── fxmanifest.lua
```
