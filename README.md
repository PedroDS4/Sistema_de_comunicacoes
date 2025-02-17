# Sistema_de_comunicacoes
Esse projeto mostra a implementação de um sistema de comunicações incluindo a modelagem do transmissor, do receptor e de alguns efeitos de canal.

##**Transmissor**
As etapas do transmissor são
* Codificação
* Filtro de pulso
* Modulação
* Transmissão ao Canal

Essas etapas serão implementadas abaixo, e sua matemática será descrita abaixo.


### **Matemática da Transmissão**
**Codificação**
A codificação é o ato de transformar os caracteres em string que irão ser transmitidos em bits, por exemplo

$$
"A"  -> 0100 0001
$$

$$
"B"  -> 0100 0010
$$

essa conversão pode ser encontrada diretamente na tabela ASCII, e existem funções que ja fazem essa conversão.


Então seja uma sequência a ser transmitida, como por exemplo abaixo

$$
Str = "Olá \ mundo"
$$

Quando codificada, gera-se um vetor com os bits dos símbolos, que depois irá ser convertido em números únicos

$$
s = letrapam(str)
$$

O tipo de codificação será uma 4-PAM
dada pela relação

00 -> -3

01 -> -1

10 ->  1

11 -> 3



**Filtro de pulso**
Depois da codificação, os símbolos dos caracteres precisam ser armazenados em um vetor, para serem transformados em um sinal analógico.

Esse vetor em que os simbolos são adicionados, pode ser representado por um trem de impulsos, dado por

$$
s[n] = \sum_{k = 0}^{M} s_k \cdot δ[n - kM]
$$


e esse sinal de simbolos é convoluído com uma forma de pulso, $p[n]$, e o sinal resultante que será transmitido, fica

$$
x[n] = s[n] \ast p[n] = \sum_{k = 0}^{M} s_k \cdot δ[n - kM] \ast p[n]
$$

então

$$
x[n] =  \sum_{k = 0}^{M} s_k \cdot (p[n] \ast \delta[n-kM]) = \sum_{k = 0}^{M} s_k p[n - kM]
$$

e o sinal $x[n]$ é passado por um conversor D/A para a modulação e posteriormente transmissão.

**Modulação**
A modulação é o ato de deslocarmos o sinal em frequẽncia, para poder transmití-la em um certo canal
Matematicamente é dada por

$$
x_{m}(t) = x(t) \cdot cos(2\pi f_c t)
$$


na frequência, o que acontece é


$$
X_m(f) = \frac{1}{2}X(f) \delta(f - fc) + \frac{1}{2}X(f) \delta(f + fc)
$$

ou seja, o espectro do sinal é deslocado para a frequência do cosseno.




##**Canal**
Depois de sair do transmissor, o sinal chega ao receptor apresentando algumas distorções de canal

**Ruído aditivo**
O ruído aditivo é a modelagem mais simplificada de um efeito de canal, onda o sinal é somado a um sinal de ruído aleatório gerado, Matematicamente temos

$$
x_{recebido}(t) = x(t) + \eta (t)
$$


onde $\eta(t)$ é um sinal de ruído qualquer, comummente representado por um ruído AWGN.


**Doppler**
O efeito dopler modifica o conteúdo de frequências transmitido, então a frequência na recepção é adicionada de uma distorção, matematicamente temos

$$
x_{recebido}(t) = x(t) \cdot cos(2 \pi (f_c + \gamma) t)
$$



**Desvanescimento**
Ao ser transmitido, o sinal pode bater em obstáculos e ter componentes de frequência atenuadas, assim diminuindo a relação sinal ruído do mesmo.
Matematicamente pode ser representado por um ganho variável na frequência ou no tempo, como segue

$$
X_{desv}(f) = A(f) \cdot X(f)
$$



##**Receptor**
Depois de passar pelo canal, o sinal chega ao receptor, onde as manipulações que foram feitas para transmití-lo precisam ser revertidas ou alteradas, para recuperar a mensagem original transmitida, além disso precisamos de algum jeito compensar os efeitos de canal.



###**Controle automático de ganho**
Depois que o sinal chega ao receptor, ele provavelmente chegou com uma amplitude diferente do que foi transmitido, e para amostrálo e quantizá-lo, precisamos adaptar o sinal a faixa dinâmica do quantizador.

Assim precisamos de um ganho $a$ que minimize a diferença entre um valor de referência que é o valor máximo do quantizador, e o sinal recebido, temos então

$$
\text{Min  } ||d - a x_r(t)||^2
$$

ou seja, minimizandoe ssa função objetivo conseguimos encontrar o melhor valor de a para que o sinal fique na faixa do valor de referência $d$.
Porém esse processamento é feito sobre o sinal $x_r(t)$ já amostrado, assim podemos minimizar uma função por otimização, e é o que será feito a seguir.

## **Derivação do algorítmo por gradiente descendente**
Definindo o sinal amostrado obtido a partir de $x_r(t)$ como $s[n]$, e definindo agora outra função objetivo, dada por

$$
J(a) = E\{\ |a| \cdot (\frac{s^2[n]}{3} - d^2) \}\ = E\{\ |a| \cdot (\frac{a^2\cdot r^2[n]}{3} - d^2) \}\
$$

derivando agora em relação a $a$, temos

$$
\frac{\partial}{\partial a}J(a) = \frac{\partial}{\partial a} E\{\ |a| \cdot (\frac{a^2\cdot r^2[n]}{3} - d^2) \}\
$$

e a derivada do valor esperado é o valor esperado das derivadas

$$
\frac{\partial}{\partial a}J(a) = E\{\ \frac{\partial}{\partial a} |a| \cdot (\frac{a^2\cdot r^2[n]}{3} - d^2) \}\
$$

utilizando a regra do produto

$$
\frac{\partial}{\partial a} |a| \cdot (\frac{a^2\cdot r^2[n]}{3} - d^2) = \frac{\partial}{\partial a} |a| \cdot (\frac{a^2\cdot r^2[n]}{3} - d^2)  + \frac{\partial}{\partial a}  (\frac{a^2\cdot r^2[n]}{3} - d^2) \cdot |a|
$$

para derivada da função módulo pode ser utilizada a função sinal, assim temos

$$
\frac{\partial}{\partial a} |a| \cdot (\frac{a^2\cdot r^2[n]}{3} - d^2)  + \frac{\partial}{\partial a}  (\frac{a^2\cdot r^2[n]}{3} - d^2) \cdot |a| = sgn(a) (\frac{a^2\cdot r^2[n]}{3} - d^2) + |a| \cdot 2a \frac{ r^2[n]}{3}
$$

e podemos escrever o módulo como sendo

$$
|a|  = sgn(a) \cdot a
$$

então finalmente temos

$$
sgn(a) (\frac{a^2\cdot r^2[n]}{3} - d^2) + |a| \cdot 2a \frac{ r^2[n]}{3} = (sgn(a) \frac{a^2\cdot r^2[n]}{3} - d^2) + \cdot sgn(a) 2a^2 \frac{ r^2[n]}{3} =  sgn(a) \cdot (a^2\cdot r^2[n] - d^2)
$$

então a função objetivo final fica

$$
\frac{\partial}{\partial a}J(a) = E \{\ sgn(a) \cdot (a^2\cdot r^2[n] - d^2)
 \}\
$$

e a atualização do gradiente é dada por

$$
a^{k+1} = a^k - \mu \frac{\partial}{\partial a}J(a^k)
$$



###**Demodulação**
A demodulação é o ato de reverter a modulação feita no transmissor, ou seja, multiplicar por outro cosseno para que surja uma componente de frequências centrada para poder ser feita a filtragem do sinal e recuperá-lo.

Seja o sinal modulado dado por
$$
x_m(t) = x(t) \cdot cos(2 \pi f_c t)
$$

para demodularmos esse sinal, multiplicamos esse sinal recebido mais uma vez por um cosseno na mesma frequência e fase, assim

$$
x_r(t) = x_m(t) \cdot cos(2 \pi f_c t) = x(t) cos^2(2 \pi f_c t)
$$

usando a seguinte identidade trigonométrica

$$
cos^2(2 \pi f_c t) = \frac{ 1 + cos(4 \pi f_c t)}{2}
$$

vemos que surge uma componente de frequência ainda mais alta, e uma réplica do espectro do sinal original na origem, então para obtermos de volta o sinal $x(t)$ basta filtrar o sinal resultante, então

$$
x(t) = 2 \cdot LPF\{\ \frac{ x(t) }{2} + \frac{ x(t) cos(4 \pi f_c t)}{2}  \}\
$$




###**Sincronização por correlação**
A operação de correlação é feita de modo a aumentar a relação sinal ruído do sinal que é recebido, que normalmente está contaminado com ruído aditivo.

A correlação entre dois sinais é dada por

$$
\phi_{xy} = \sum_{k = -∞ }^{∞} x[k] y[n + k]
$$


que também pode ser vista como uam convolução entre o sinal x e o sinal y refletido, uma vez que

$$
\sum_{k = -∞ }^{∞} x[k] y[n + k] = \sum_{k = -∞ }^{∞} x[k] y[-(-n - k)]
$$

e se temos

$$
y_r[n] = y[-n]
$$

então

$$
\phi_{xy} = \sum_{k = -∞ }^{∞} x[k] y_r[n - k] = x[n] \ast y[-n]
$$

muitas vezes a sincronização é implementada assim, e essa implementação também pode ser chamada de filtro casado(Match Filter).
