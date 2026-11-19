# Rádio Brasil FM

Aplicativo Android de rádios brasileiras: gratuito, **sem anúncios**, sem conta e sem rastreamento.
Catálogo vindo do diretório comunitário [Radio Browser](https://www.radio-browser.info), com cache offline e atualização diária.

> **Status honesto:** este código foi escrito mas **ainda não foi compilado nem testado em um aparelho**.
> A primeira compilação pode revelar erros pequenos (versões de bibliotecas, imports). Se o build falhar, copie a mensagem de erro do GitHub Actions e peça a correção.

## Como obter o APK (sem instalar nada no PC)

1. Crie uma conta gratuita em https://github.com.
2. Crie um repositório novo (privado serve): **New repository**.
3. Envie **todo o conteúdo desta pasta** para o repositório (**Add file → Upload files**, arraste os arquivos e pastas).
   - Confira que existe `.github/workflows/build-apk.yml`. Pastas que começam com ponto costumam ficar ocultas no computador. Se não subir, use **Add file → Create new file**, digite o nome `.github/workflows/build-apk.yml` e cole o conteúdo do arquivo.
4. Abra a aba **Actions**. O workflow **Compilar APK** roda sozinho a cada envio (ou clique em **Run workflow**). Leva de 5 a 10 minutos.
5. Ao terminar (marca verde), abra a execução e baixe o arquivo **RadioBrasilFM-apk** na seção *Artifacts*. É um ZIP com o `app-debug.apk`.
6. Se aparecer marca vermelha, abra o passo que falhou e copie o erro.

### Alternativa: Android Studio
Abra esta pasta no Android Studio (versão recente), aguarde a sincronização do Gradle e use **Build → Build APK(s)**.
O APK fica em `app/build/outputs/apk/debug/app-debug.apk`.

## Instalar no celular
1. Passe o `app-debug.apk` para o celular (cabo, Drive, e-mail).
2. Toque no arquivo e permita **instalar apps desta fonte** quando o Android pedir.
3. Abra o app e permita as notificações (necessárias para os controles de mídia).

O APK é assinado com a chave de *debug*, suficiente para uso pessoal. Para publicar na Play Store seria preciso um build *release* assinado com sua própria chave.

## Roteiro de teste manual no celular
- Abrir o app: o catálogo carrega (barra de progresso) e as seções aparecem.
- Buscar por nome, estado e categoria; combinar filtros.
- Tocar numa rádio: ela toca e aparece o mini player e a notificação.
- Pausar e retomar pelo mini player, pela notificação e por fone Bluetooth.
- Bloquear a tela e deixar tocando; deslizar o app da lista de recentes (a rádio continua).
- Ativar o modo avião: aparece "Reconectando…" e depois "Emissora indisponível" com **Tentar novamente**. Desativar o modo avião: a reprodução volta sozinha.
- Favoritar, reordenar (setas) e fechar/abrir o app: os dados continuam.
- Temporizador: escolher 15 min (para teste, confira a contagem regressiva) e ver se a rádio para no fim.
- Ajustes: tema claro/escuro, região preferida, atualizar catálogo.
- Abrir sem internet depois de já ter carregado uma vez: usa o catálogo salvo.

## Testes automatizados
`gradle testDebugUnitTest` (o workflow já roda): filtros e busca, mapeamento e validação do catálogo, reconhecimento de estados e temporizador.
Não há testes instrumentados (reprodução real, segundo plano, reconexão): esses estão no roteiro manual acima.

## Arquitetura
Kotlin + Jetpack Compose (Material 3), MVVM em três camadas:
- `domain/`: modelos, categorias, estados brasileiros e filtro de busca (lógica pura, testável).
- `data/`: catálogo (OkHttp + Gson, cache em arquivo), DataStore (favoritos, histórico, ajustes), WorkManager (atualização diária).
- `player/`: `PlaybackService` (Media3 ExoPlayer + MediaSession, serviço em primeiro plano `mediaPlayback`), `PlayerController` (ponte com a interface), `SleepTimer`.
- `ui/`: telas Compose (Início, Buscar, Favoritas, Ajustes, mini player e player completo).

### Decisões técnicas
- **Sem Hilt, Room e Retrofit:** o projeto é pequeno e eu não consigo compilar aqui. Menos plugins (KSP/kapt) significa menos risco de erro de build. Injeção manual em `RadioApp.kt`, cache em arquivo JSON e OkHttp direto.
- **HTTP:** muitas rádios brasileiras só têm HTTP. O app aceita HTTP **apenas** para os streams (`network_security_config.xml`). A API do catálogo usa HTTPS obrigatório (`ConnectionSpec.MODERN_TLS`). A validação TLS nunca é desativada.
- **Reconexão:** até 6 tentativas com espera crescente; também tenta ao voltar a internet. Formatos não suportados falham direto.
- **Sem anúncios:** nenhum SDK de publicidade ou analytics. Permissões usadas: internet, estado da rede, serviço em primeiro plano, notificações e wake lock (mantém a rede ativa durante a reprodução).

## Custos se você precisar de servidor próprio
Hoje não há servidor: o app consulta o Radio Browser (gratuito, sem garantia de disponibilidade). Um backend próprio só valeria a pena para curar e testar as rádios periodicamente: uma VPS pequena custa cerca de US$ 5 a 10 por mês, mais o tempo de manutenção. Verificar milhares de streams exige CPU e banda extras.

## Limitações conhecidas
- O Radio Browser não tem campo de **cidade**: a busca por cidade só funciona se o nome da cidade aparecer no nome, no estado ou nas tags. Há filtro por estado.
- Categorias são deduzidas das tags e do nome; emissoras sem tags ficam sem categoria. Rádios comunitárias, universitárias e públicas dependem de a emissora usar essas palavras.
- Dados da comunidade: há streams fora do ar, duplicados ou com nome/logo errados.
- "Informar problema" oculta a emissora só neste aparelho (não há servidor para receber relatórios).
- Não envia "clique" ao Radio Browser (ranking deles); só consulta.
- O título da música (ICY) aparece só se a emissora enviar.
- Reordenação de favoritos por setas (sem arrastar).
- Sem Android Auto/Chromecast, sem gravação, sem equalizador.
- Se a rede cair e o app for fechado, pode ser preciso tocar de novo.

## Melhorias possíveis
Catálogo próprio curado; Android Auto; arrastar para reordenar; widget; busca por cidade com base de municípios; exportar/importar favoritos; testes instrumentados.

## Fontes e termos
Catálogo: Radio Browser (API pública e gratuita; envia-se um User-Agent próprio). Cada transmissão pertence à sua emissora; o app só reproduz os streams públicos que elas disponibilizam e não remove os anúncios que estiverem na transmissão original.
