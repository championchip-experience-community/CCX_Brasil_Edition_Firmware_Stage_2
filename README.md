# Guia de Integração de Firmware

Este repositório fornece o firmware utilizado para validar o seu processador ChampionCHIP.

Existem duas formas recomendadas de utilizá-lo:

1. **Mock Simples** — para um teste rápido no OpenLane.
2. **Firmware Completo** — para simulação e validação do processador exigidas no Desafio.

---

## 1. Mock Simples — Teste no OpenLane

Para a sua execução no OpenLane, você **não precisa sintetizar o firmware completo**.

O firmware completo contém muitas instruções e implementar todas elas como um grande bloco `case` pode gerar células desnecessárias.

Em vez disso, crie um bloco pequeno de Memória de Instruções (Instruction Memory):

```verilog
module mock_imem (
    input  wire [31:0] i_address,
    output reg  [31:0] o_instruction
);

always @(*) begin
    case (i_address)

        32'h00400000: o_instruction = 32'h00A00293; // addi x5, x0, 10
        32'h00400004: o_instruction = 32'h00500313; // addi x6, x0, 5
        32'h00400008: o_instruction = 32'h006283B3; // add  x7, x5, x6

        default:
            o_instruction = 32'h00000013; // NOP

    endcase
end

endmodule
```

Esse programa simples executa:

```text
x5 = 10
x6 = 5
x7 = x5 + x6
```

Resultado esperado:

```text
x7 = 15
```

---

## 2. Firmware Completo — Validação do Processador

Para validação funcional, utilize o arquivo `firmware.txt` completo disponibilizado neste repositório.

O firmware inicia em:

```text
0x00400000
```

Portanto, o seu processador deve iniciar a execução em:

```verilog
PC = 32'h0040_0000;
```

### Utilizando o Firmware no ChipInventor Digital

Crie um bloco Verilog para atuar como a sua **Memória de Instruções (IMEM)**.

O bloco recebe o endereço da instrução vindo do seu processador e retorna a instrução correspondente:

```verilog
module firmware_memory (
    input  wire [31:0] i_address,
    output reg  [31:0] o_instruction
);

always @(*) begin
    case (i_address)

        32'h00400000: o_instruction = 32'h123452B7;
        32'h00400004: o_instruction = 32'h12345337;
        32'h00400008: o_instruction = 32'h3E629463;
        32'h0040000C: o_instruction = 32'h00001297;
        32'h00400010: o_instruction = 32'h3E028063;
        32'h00400014: o_instruction = 32'h00A00293;
        32'h00400018: o_instruction = 32'hFFD28313;
        32'h0040001C: o_instruction = 32'h00700393;

        // Continue com as instruções restantes do firmware

        default:
            o_instruction = 32'h00000013; // NOP

    endcase
end

endmodule
```

> **Importante:** O código acima contém apenas as primeiras instruções como exemplo. Para testar o **processador completo**, copie **todas as linhas de endereço/instrução do `firmware.txt` fornecido** para dentro da estrutura `case` do seu bloco IMEM de Firmware.

Conecte o bloco ao seu processador:

```text
Endereço de Instrução do Processador
                 |
                 v
        +-----------------+
        |  IMEM Firmware  |
        +-----------------+
                 |
                 v
     Instrução para o Processador
```

### Verificando o Resultado

Ao final da execução do firmware, o registrador `x4` indica o resultado:

```text
x4 = 0x00000000   -> SUCESSO (PASS)
x4 = 0xFFFFFFFF   -> FALHA (FAIL)
```

Monitore o registrador `x4` durante a simulação para verificar a execução.

> **Lembre-se:** use o **mock simples para um teste rápido no OpenLane** e copie o **`firmware.txt` completo** para o seu bloco IMEM para a validação integral do processador.
