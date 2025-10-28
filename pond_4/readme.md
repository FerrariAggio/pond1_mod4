# Projetos Arduino

Este repositório contém o código desenvolvido para a atividade de Arduino utilizando programação OO e ponteiro de memória.

### 🔗 [Acesse o vídeo de demonstração](https://drive.google.com/file/d/17siPUWbCxeSCKZl-o9sXgJn77c3qHNFp/view?usp=drive_link)

## Materiais utilizados

| Material              | Quantidade |
| ---------------------- | ---------- |
| Arduino Uno            | 1          |
| Protoboard             | 1          |
| Led                    | 3          |
| Resistor               | 3          |
| Flet (Macho/Macho)     | 6          |
| Flet (Macho/Fêmea)     | 1          |



## Código desenvolvido

Abaixo é possível visualizar a código realizado para o funcionamento do sistema, nele foram configurado a classe de um Led e as funções para liga-lo e desliga-lo.

```C++ 
class Led {
  int* port_;

public:
  Led(int* port) {
    port_ = port;
    pinMode(*port_, OUTPUT);
  }

  void On() {
    digitalWrite(*port_, HIGH);
  }

  void Off() {
    digitalWrite(*port_, LOW);
  }
};

int portaVermelho = 13;
int portaAmarelo  = 12;
int portaVerde    = 11;

Led vermelho(&portaVermelho);
Led amarelo(&portaAmarelo);
Led verde(&portaVerde);

void setup() {
}

void loop() {
  vermelho.Off();
  amarelo.Off();
  verde.On();
  delay(4000);
  vermelho.Off();
  amarelo.On();
  verde.Off();
  delay(2000);
  vermelho.On();
  amarelo.Off();
  verde.Off();
  delay(6000);
}

```