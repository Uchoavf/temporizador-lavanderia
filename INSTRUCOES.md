# Aplicativo: Cronômetro de Lavanderia

## Visão Geral
Single Page Application (HTML + CSS + JS) para controle de uso de máquinas de lavar e secar. Funciona 100% no navegador, sem servidor backend. Os dados são salvos no **localStorage** do navegador.

## Funcionalidades
- Múltiplas **lavanderias** (locais), cada uma com suas próprias máquinas
- Duas **colunas** de máquinas: Lavadoras e Secadoras
- **Temporizador independente** por máquina, todos rodando simultaneamente
- Presets de tempo (15min, 30min, 45min, 60min, 90min, 120min) + customizado
- Barra de progresso visual
- **Histórico** em tempo real: registra início, conclusão e cancelamento
- Relógio com data/hora automática
- Design responsivo (mobile-first)

## Estrutura de Arquivos
```
Temporizador de Lavanderia/
  index.html           -> Aplicativo principal
  INSTRUCOES.md        -> Este arquivo
```

## Arquitetura

### Chaves do localStorage
Todas as chaves são prefixadas com `lav2_`:
- `lav2_laundries` -> Array `[{ id, name }]`
- `lav2_machines` -> Objeto `{ [laundryId]: [{ id, name, type }] }` onde `type` é `'lavadora'` ou `'secadora'`
- `lav2_timers` -> Objeto `{ [machineId]: { remaining, total, running, endAt } }`
- `lav2_history` -> Array `[{ id, date, laundryName, machineName, machineType, duration, status }]` onde `status` é `'iniciado'`, `'completo'` ou `'cancelado'`
- `lav2_selected` -> `{ laundryId }`

### Motor de Temporizadores
- Ao iniciar, o timer grava `endAt` (horário de término); o tempo restante é sempre calculado a partir dele, então não perde tempo com tela bloqueada, aba em segundo plano ou página recarregada
- Um `setInterval` global (500 ms) chama `tick()`: finaliza timers vencidos (de qualquer lavanderia) e atualiza só o texto/barra dos cards em execução
- Ao terminar: registra "completo" no histórico e dispara alarme (som, vibração e notificação, se permitida)
- O áudio e a permissão de notificação são liberados no primeiro clique em "Iniciar"

### Funções principais (JavaScript)
| Função | Descrição |
|--------|-----------|
| `toggleTimer(id)` | Inicia/Pausa o timer de uma máquina |
| `resetMachineTimer(id)` | Reseta o timer de uma máquina |
| `setTimerMinutes(id, min)` | Define o tempo total de uma máquina |
| `addHistoryEntry(id, status)` | Registra evento no histórico |
| `tick()` / `completeExpired()` | Loop global: finaliza timers vencidos e atualiza a tela |
| `getRemaining(t)` | Tempo restante calculado a partir de `endAt` |
| `alertFinished(ids)` | Alarme sonoro, vibração e notificação |
| `renderMachines()` | Renderiza a lista de máquinas em colunas |
| `renderHistory()` | Renderiza o histórico |
| `renderMachineCard(m, timers)` | Renderiza o card de uma máquina |

### Fluxo de uso
1. Usuário cria/ seleciona uma **lavanderia**
2. Adiciona **máquinas** com nome e tipo (lavadora/secadora)
3. Cada máquina exibe seu próprio card com timer, presets e controles
4. Pode-se iniciar múltiplos timers simultaneamente
5. Ao iniciar -> histórico registra "Iniciado" (pausar e retomar não gera nova entrada)
6. Ao completar -> histórico registra "Completo" + alarme sonoro
7. Ao resetar enquanto roda -> histórico registra "Cancelado"

## Como servir
```bash
python3 -m http.server 8080 -d "/caminho/para/Temporizador de Lavanderia"
```
Depois acessar `http://SEU_IP:8080` no celular (mesma rede Wi-Fi).

## Personalização
- Para adicionar novos presets de tempo, edite os `<button data-minutes="...">` no HTML
- Cores e temas podem ser alterados no bloco `<style>`
- O tipo legado `'ambos'` é convertido para `'lavadora'` ao carregar

## Backup
O histórico do git é o backup do código; os dados do usuário ficam só no `localStorage` do navegador.
