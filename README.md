# Programando o LEGO Mindstorms NXT em NXC (Linux)

Tutorial passo a passo: da instalação do compilador ao primeiro robô autônomo.

Este repositório documenta como programar o robô **LEGO Mindstorms NXT** utilizando **NXC** (*Not eXactly C*), uma linguagem textual com sintaxe parecida com C, como alternativa ao ambiente gráfico oficial (NXT-G), em ambiente **Ubuntu Linux**.

## Sumário

- [Pré-requisitos](#pré-requisitos)
- [1. Instalação do compilador NXC (nbc) no Ubuntu](#1-instalação-do-compilador-nxc-nbc-no-ubuntu)
- [2. Conectando o NXT ao computador](#2-conectando-o-nxt-ao-computador)
- [3. Escrevendo o primeiro programa](#3-escrevendo-o-primeiro-programa)
- [4. Compilando e enviando o programa para o robô](#4-compilando-e-enviando-o-programa-para-o-robô)
- [5. Programa avançado: robô desviador de obstáculos](#5-programa-avançado-robô-desviador-de-obstáculos)
- [Estrutura do repositório](#estrutura-do-repositório)

---

## Pré-requisitos

- Computador com Ubuntu Linux (ou outra distribuição baseada em Debian)
- Robô LEGO Mindstorms NXT montado, com bateria carregada
- Cabo USB (o mesmo que acompanha o kit)
- Acesso ao terminal com permissão `sudo`
- Um editor de código (recomendado: Visual Studio Code)

---

## 1. Instalação do compilador NXC (nbc) no Ubuntu

O compilador `nbc`, que entende a linguagem NXC, está disponível diretamente nos repositórios oficiais do Ubuntu:

```bash
sudo apt update
sudo apt install nbc
```

Para confirmar que a instalação funcionou:

```bash
nbc -help
```

Se aparecer uma lista de opções do compilador na tela, a instalação foi concluída com sucesso.

> **Observação:** caso o pacote `nbc` não seja encontrado (em versões muito recentes do Ubuntu ele pode ter sido removido dos repositórios padrão), será necessário compilar a partir do código-fonte, usando `build-essential` e `libusb-1.0-0-dev`.

---

## 2. Conectando o NXT ao computador

Ligue o tijolo NXT e conecte o cabo USB ao computador. Verifique se o Linux reconheceu o dispositivo:

```bash
lsusb | grep -i lego
```

O retorno esperado é algo como `ID 0694:0002 Lego Group Mindstorms NXT`.

Se o envio do programa falhar por erro de permissão, crie uma regra de acesso USB:

```bash
sudo nano /etc/udev/rules.d/70-lego.rules
```

Cole esta linha dentro do arquivo:

```
SUBSYSTEM=="usb", ATTR{idVendor}=="0694", MODE="0666"
```

Salve (`Ctrl+O`, `Enter`, `Ctrl+X`) e recarregue as regras:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## 3. Escrevendo o primeiro programa

Crie um arquivo chamado exatamente `teste.nxc` (a extensão `.nxc` é obrigatória — veja o arquivo completo em [`src/teste.nxc`](src/teste.nxc)):

```c
task main()
{
    ClearScreen();
    TextOut(0, LCD_LINE1, "Ola, NXT!");
    TextOut(0, LCD_LINE3, "Testando motores");

    PlayTone(440, 500);
    Wait(500);

    OnFwd(OUT_B, 50);
    OnFwd(OUT_C, 50);

    Wait(2000);

    Off(OUT_B);
    Off(OUT_C);

    PlayTone(660, 300);
    Wait(300);

    TextOut(0, LCD_LINE5, "Fim do teste!");
}
```

**O que esse programa faz:**
- Limpa o visor e mostra duas mensagens de texto
- Toca um som (440 Hz por 500 ms)
- Liga os motores das portas B e C (rodas) a 50% de potência por 2 segundos
- Desliga os motores e toca um segundo som, avisando o fim do teste

⚠️ **Salve o arquivo (`Ctrl+S`) antes de compilar.** Esquecer esse passo é uma causa comum do erro `Error: No task named "main" exists`, mesmo com o código certo na tela.

---

## 4. Compilando e enviando o programa para o robô

Com o NXT ligado e conectado via USB, vá até a pasta do arquivo pelo terminal e execute:

```bash
nbc teste.nxc -d
```

A opção `-d` (download) compila o código e já envia o programa compilado diretamente para o tijolo via USB. No visor do NXT, vá em **My Files → Software Files**, selecione o programa **teste** e aperte o botão laranja para executá-lo.

---

## 5. Programa avançado: robô desviador de obstáculos

Depois de validar o funcionamento básico, o próximo passo é usar um sensor para dar autonomia ao robô. O programa abaixo usa o **sensor ultrassônico**, conectado na porta 1, para detectar obstáculos e desviar automaticamente. Arquivo completo em [`src/robo_desviador.nxc`](src/robo_desviador.nxc).

```c
task main()
{
    SetSensorLowspeed(IN_1);      // Sensor ultrassonico na porta 1
    ClearScreen();
    TextOut(0, LCD_LINE1, "Robo Desviador");
    TextOut(0, LCD_LINE2, "Iniciando...");
    PlayTone(523, 300);
    Wait(1000);

    int distancia;
    int distanciaMinima = 20;     // Distancia minima em cm para desviar

    while (true)
    {
        distancia = SensorUS(IN_1);

        ClearScreen();
        TextOut(0, LCD_LINE1, "Distancia:");
        NumOut(0, LCD_LINE2, distancia);

        if (distancia < distanciaMinima && distancia > 0)
        {
            TextOut(0, LCD_LINE4, "Obstaculo!");
            PlayTone(300, 200);

            Off(OUT_B);
            Off(OUT_C);
            Wait(200);

            OnRev(OUT_B, 50);
            OnRev(OUT_C, 50);
            Wait(500);
            Off(OUT_B);
            Off(OUT_C);

            OnFwd(OUT_B, 50);
            OnRev(OUT_C, 50);
            Wait(600);
            Off(OUT_B);
            Off(OUT_C);
        }
        else
        {
            TextOut(0, LCD_LINE4, "Livre - Andando");
            OnFwd(OUT_B, 60);
            OnFwd(OUT_C, 60);
        }

        Wait(100);
    }
}
```

> Se o sensor ultrassônico estiver em outra porta, troque `IN_1` pela porta correta nas duas linhas onde ele aparece.

**Funcionamento passo a passo:**
1. O sensor ultrassônico mede a distância até objetos à frente, continuamente
2. A distância medida é exibida em tempo real no visor do NXT
3. Se a distância for menor que 20 cm, o robô para, toca um som de alerta, dá ré e gira para desviar
4. Se o caminho estiver livre, o robô segue em frente normalmente
5. Todo o processo roda em loop infinito (`while(true)`), fazendo o robô reagir ao ambiente continuamente

---

## Estrutura do repositório

```
.
├── README.md
└── src/
    ├── teste.nxc            # Programa de teste inicial (motores, som, visor)
    └── robo_desviador.nxc   # Programa avançado com sensor ultrassônico
```
