    # AURUM ROBOTICS - Para acompanhar imagens acesse o instagram @spirit.aurum 

OBS: Esse é o meu primeiro repositório, já acessei alguns mas nunca criei nada para compartilhar com a comunidade, então não sei muito bem como isso vai funcionar ou se alguém vai acabar me ajudando com minhas dificuldades ao longo do projeto,
então aceito qualquer dica sobre como melhorar.

### Esse é o projeto de um robo com esteira lagarta e câmera + um controle remoto com multiplas funções, então vou dividir a minha explicação nos dois tópicos principais, ESTEIRA e CONTROLE

### Componentes que já tenho em mãos:
    ## ESTEIRA
      Caracteristicas da esteira;
      - 2 motores DC 130 com caixa de redução (controlam as esteiras)
      - 1 chassi estilo tanque esteira lagarta
      - 1 Placa ESP32-S3 CAM - ESP32-S3 N16R8 - 80V564
      - 1 Módulo Driver Ponte H - L298N
      - Suporte Pan Tilt Fpv Arduino Drone Pic Com 2 Servos Sg90
  

     ## COONTROLE
      Caracteristicas do controle;
      - 1 Placas DOIT ESP32 - CH9102
      - 1 Display TFT 2.4 SPI - Touch Screen - ST7789V
      - 2 Módulo Joystick Analógico 3 Eixos 5v Ky-023 Arduino
 

    Materiais extras;
       - 1 Placas DOIT ESP32 - CH9102
       - Algumas protoboards de tamanhos variados
       - Caneta de impressão 3D
       - 2 motores servo sg90
      Dentre outros itens que aparecem na foto.

  ## Objetivo de cada tópico:

## Esteira
-   Obetivo inicial é poder ser controlada pelo controle e conseguir movimentar a câmera pelo pantilt visualizando as imagens remotamente
Objetivos futuros é que a esteira possa receber upgrades e atualizações de código remotamente, sendo enviado direto pelo controle conectado.

## Esteira
- Controlar a esteira e qualquer outra esteira que seja criada posteriormente, poder conectar o controle no WIFI para baixar atualização e melhoria de codigos, tanto para o controle quanto para a esteira.
No display TFT poder visualizar as imagens da câmera e também conseguir navegar em um menu de opções e funções.

## Código que criei para o controle:
Esse código tem algum problema na questão da seleção do menu, ele funciona bem até a parte de abrir o menu, mas após abrir ele fica trocando a seleção sozinho sem parar. Não sei bem como resolver isso, não é nada de hardware, fiz alguns testes, mas infelizmente continuou o erro.
#CODIGO para rodar no ARDUINO IDE - Modelo ESP do controle  ESP32-D0WD-V3:

      //Aurum Robotics, 04/2025
      // Controle de esteira utilizando ESP32, TFT e XPT2046
      // Este código implementa um menu de controle para uma esteira, utilizando um display TFT e um touchscreen resistivo.
      // O menu permite conectar a esteira, atualizar o controle e atualizar a esteira. O controle é feito através de dois joysticks e um botão de toque.
      // Problema atual com a funcionalidade do menu, onde a seleção fica alternando sozinha, mesmo sem interação do usuário.


      #include <TFT_eSPI.h>  // Biblioteca do display
      #include <XPT2046_Touchscreen.h>  // Biblioteca do touch resistivo

      // Configuração do display
      TFT_eSPI tft = TFT_eSPI();

      // Configuração do touch (CS na GPIO 21)
      #define TOUCH_CS 21
      XPT2046_Touchscreen ts(TOUCH_CS);

      // Pinos dos Joysticks
      #define VRX_DIREITO 34
      #define VRY_DIREITO 32
      #define SW_DIREITO 25
      #define VRX_ESQUERDO 35
      #define VRY_ESQUERDO 33
      #define SW_ESQUERDO 26

      // Estados do menu
      int menuIndex = 3; // Inicia com a opção "Fechar menu" selecionada
      bool menuAberto = false;

      // Opções do menu
      const char *opcoesMenu[] = {"Conectar esteira", "Atualizar controle", "Atualizar esteira", "Fechar menu"};
      const int totalOpcoes = 4;

      // Variáveis para controle do analógico
      const int limiteSuperior = 3900; // Valor máximo do ADC para cima
      const int limiteInferior = 200;  // Valor mínimo do ADC para baixo
      unsigned long ultimoMovimento = 0;
      const int intervaloMovimento = 200; // Tempo mínimo entre movimentações
      bool aguardandoCentro = false;

      void setup() {
    Serial.begin(115200);
    tft.init();
    tft.setRotation(1);
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.setTextSize(2);
    
    pinMode(SW_DIREITO, INPUT_PULLUP);
    pinMode(SW_ESQUERDO, INPUT_PULLUP);
    
    ts.begin();
    ts.setRotation(1);
    
    desenharTelaInicial();
      }

      void loop() {
    if (menuAberto) {
        navegarMenu();
    } else {
        if (digitalRead(SW_DIREITO) == LOW) {
            delay(200);  // Debounce
            if (digitalRead(SW_DIREITO) == LOW) {
                menuAberto = true;
                desenharMenu();
            }
        }
    }
      }

      void desenharTelaInicial() {
    tft.fillScreen(TFT_BLACK);
    tft.setTextSize(2);
    int16_t x = (tft.width() - 14 * 12) / 2;
    int16_t y = (tft.height() - 24) / 2;
    tft.setCursor(x, y);
    tft.print("PROCURANDO VIDEO");
      }

      void desenharMenu() {
    tft.fillScreen(TFT_NAVY);
    tft.setTextSize(2);
    for (int i = 0; i < totalOpcoes; i++) {
        int16_t x = 20;
        int16_t y = 50 + i * 40;
        tft.fillRect(x - 5, y - 5, tft.width() - 40, 30, TFT_BLACK); // Caixa de seleção
        if (i == menuIndex) {
            tft.drawRect(x - 5, y - 5, tft.width() - 40, 30, TFT_YELLOW);
            tft.setTextColor(TFT_YELLOW, TFT_BLACK);
        } else {
            tft.setTextColor(TFT_WHITE, TFT_BLACK);
        }
        tft.setCursor(x, y);
        tft.print(opcoesMenu[i]);
    }
      }

      void navegarMenu() {
    int valorAnalogico = analogRead(VRY_DIREITO);
    unsigned long tempoAtual = millis();

    if (aguardandoCentro) {
        if (valorAnalogico > limiteInferior && valorAnalogico < limiteSuperior) {
            aguardandoCentro = false;
        }
        return;
    }

    if (tempoAtual - ultimoMovimento > intervaloMovimento) {
        if (valorAnalogico >= limiteSuperior) {
            menuIndex = (menuIndex + 1) % totalOpcoes;
            desenharMenu();
            ultimoMovimento = tempoAtual;
            aguardandoCentro = true;
        } 
        else if (valorAnalogico <= limiteInferior) {
            menuIndex = (menuIndex - 1 + totalOpcoes) % totalOpcoes;
            desenharMenu();
            ultimoMovimento = tempoAtual;
            aguardandoCentro = true;
        }
    }
      }

      void abrirSubmenu(const char *titulo) {
    tft.fillScreen(TFT_BLACK);
    tft.setTextSize(2);
    int16_t x = (tft.width() - strlen(titulo) * 10) / 2;
    int16_t y = (tft.height() - 24) / 2;
    tft.setCursor(x, y);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.print(titulo);
    
    desenharBotaoVoltar();
    
    while (true) {
        int valorAnalogico = analogRead(VRY_DIREITO);
        if (valorAnalogico >= limiteSuperior || valorAnalogico <= limiteInferior) {
            desenharMenu();
            return;
        }
        
        if (ts.touched()) {
            TS_Point p = ts.getPoint();
            int xTouch = map(p.x, 200, 3800, 0, tft.width());
            int yTouch = map(p.y, 200, 3800, 0, tft.height());
            if (xTouch >= 80 && xTouch <= 240 && yTouch >= 200 && yTouch <= 240) {
                desenharMenu();
                return;
            }
        }
    }
      }

      void desenharBotaoVoltar() {
    tft.fillRect(80, 200, 160, 40, TFT_RED);
    tft.setTextSize(2);
    tft.setTextColor(TFT_WHITE, TFT_RED);
    tft.setCursor(120, 215);
    tft.print("VOLTAR");
      }


## A esteira não tive progresso com o código, basicamente não consigo fazer ela criar uma rede wifi com seus controles, muito menos conectar em uma rede wifi para eu controlar ele pelo WEB server como testes inicial.
Modelo do ESP utilizado - ESP32 S3:
## Código da esteira:

      SEM CÓDIGO OFICIAL AINDA, ACEITO SUGESTÕES
 
 **Pull Request**!
