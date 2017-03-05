# Operational-Systems
Processos, Threads, CPUs e Escalonamento

LAB2
Parte1
Seu shell precisa de um identificador do prompt. Em minha máquina, o shell padrão usa o identificador abaixo:

| ~ @ userName (yourUser)
| => 

Altere o programa xeu.c para criar um identificador para o seu prompt. No mínimo, seu prompt deve seguir a forma abaixo:

| $username => 

Onde $username deve ser substituído pelo seu nome de usuário.

Parte2
Seu shell foi feito para executar programas. Modifique o código fonte enviado na parte1 de maneira que seja possível executar 
programas com ou sem argumentos. Somente um programa pode ser especificado pelo usuário por cada vez. O usuário poderá dar 
entrada a um novo programa no prompt após o fim da execução do programa anterior. A saída dos programas, quando houver, 
deverá ser feita na saída padrão. Dois programas possíveis seriam:

| ~ @  userName (yourUser)
| =>  date
Sex 18 Nov 2016 16:36:43 BRT

| ~ @  userName (yourUser)
| =>  sleep 5

Parte3
Quem não se comunica … Pipes são bastante usados interprocess communication (IPC) no shell.
Até agora seu shell não dá suporte a pipes (p.ex tente executar `ps xau | grep root`). 
Nessa parte do laboratório, você precisará implementar o suporte a pipes.
Parte4
Seu shell pode ter memória. Modifique o código fonte enviado na parte3 para mostrar o histórico de execuções passadas. 
Para ver o histórico, o usuário dará entrada no prompto com o comando `track`. A saída desse comando deverá ser da seguinte forma:

index	nome_do_programa_e_args pid uid gid start_epoch end_epoch return_value

Onde:
index é um número que indica que a ordem de execução dos programas nessa instância do seu shell
nome_do_programa_e_args indica a entrada passada pelo usuário
pid registra o identificador do processo associado com a execução do programa
uid registra o identificador do usuário que executou o programa
gid registra o identificador do grupo do usuário que executou o programa
start_epoch indica o unix epoch time do momento em que o programa começou a executar 
end_epoch indica o unix epoch time do momento em que o programa terminou de executar 
return_value indica o valor de retorno do processo associado com o programa
================================================================================================
LAB3
perf
O programa perf permite instrumentar eventos de software e hardware em sua máquina. Entre esses eventos, temos: 
número de ciclos de CPU, número de misses de cache, número de migrações de CPU, entradas e retornos de system calls e eventos TCP.

Para verificar quais eventos podem ser instrumentados, execute:
perf list

Os eventos são agrupados. Por exemplo, para lista os eventos específicos de escalonamento, execute:
	perf list 'sched:*'

cpu_burn.c

1 #include <math.h>
2
3 int main() {
4
5     while(1) {
6         pow(1.2, 2.3);
7     }
8     return 0;
9 }

Roteiro
Parte 1
Altere o programa cpu_burn.c para que ele execute a função pow número pré-definido de vezes. 
Esse número precisa ser escolhido de maneira tal que o programa execute por cerca de 1 segundo.

Instrumente a execução do seu programa através do comando perf dessa maneira:
perf sched record ./cpu_burn

Ao executar o comando acima, perf irá salvar os resultos no arquivo perf.data. 
Para analisar a instrumentação, execute o comando abaixo:
			perf sched latency
------------------------------------------------------------------
Parte2  LAB3
Na segunda parte desse laboratório execute múltiplas instâncias do seu programa cpu_burn.
Serão três modos de execução diferentes, a depender do número de instâncias: 
i) uma instância; 
ii) igual ao número de processadores na máquina; e 
iii) dobro do número de processadores em sua máquina. 
Faça um plot no qual o eixo x tenha o número de instância considerado em cada modo e no eixo y mostre as variáveis de saída
do comando perf sched latency. 
Execute cada modo 15 vezes.

====================================================================================
Políticas de escalonamento

As políticas de escalonamento do kernel decidem o quê (qual thread) irá executar e quando será essa execução.

Muitas políticas para kernels UNIX tomam como base o conceito de prioridade. 
Processos com maior prioridade devem executar antes de processo de menor prioridade. Ainda, 
no linux, processos com maior prioridade executam por um período mais longo do que processo com menor prioridade. Nesse kernel, 
é possível alterar a prioridade de um processo através do comando nice. Valores maiores de nice indicam menor prioridade.

O kernel do linux implementa três políticas "normais" (SCHED_OTHER, SCHED_IDLE, SCHED_BATCH) e 
duas políticas real-time (SCHED_FIFO, SCHED_RR). Nesse laboratório iremos estudar experimentalmente as políticas real-time.

http://man7.org/linux/man-pages/man2/sched_setscheduler.2.html 
http://man7.org/linux/man-pages/man7/sched.7.html 
----------------------------
Roteiro
Altere o programa cpu_burn.c, calibrado no laboratório 3, para que ele seja executado com uma política de escalonamento específica.
Deve ser possível executar o programa com duas políticas: FIFO e RR. 
O programa pode receber como argumento 0, 1 e 2. O argumento 0 indica que você não deve mudar a política de escalonamento, 
enquanto que os argumentos 1 e 2 indicam a escolha da política FIFO e RR, respectivamente.

Para especificar a política de escalonamento use as seguintes declarações:

struct sched_param param;
param.sched_priority = 99;
sched_setscheduler (0, SCHED_FIFO, &param);//SCHED_FIFO ou SCHED_RR

Você executará múltiplas instâncias do programa cpu_burn em dois modos distintos, com base no número N de CPUs da máquina
usada nos experimentos:
N/2 instâncias vanilla (sem mudança da política de escalonamento) e N/2 instâncias que usam a políticas FIFO;
N/2 instâncias vanilla (sem mudança da política de escalonamento) e N/2 instâncias que usam a políticas RR;

execute a função pow número pré-definido de vezes. Esse número precisa ser escolhido de maneira tal que o 
programa execute por cerca de 1 segundo.

Instrumente a execução do seu programa através do comando perf dessa maneira:
perf sched record ./cpu_burn $arg

Ao executar o comando acima, perf irá salvar os resultados no arquivo perf.data. 
Para analisar a instrumentação, execute o comando abaixo:
			perf sched latency

Faça um plot no qual o eixo x tenha o número de instância considerado em cada modo e no eixo y 
mostre as variáveis de saída do comando perf sched latency. Execute cada modo 15 vezes.


implemente script run_lab4.sh que roda o seu experimento. 
Esse script deve receber um argumento: FIFO ou RR, respectivamente para os modos 1 e 2, 
descritos acima. Além disso,faça um script plot_lab4.sh que plota o gráfico pedido na descrição acima. 
Esse script deve receber um diretório como parâmetro (nesse diretório devem estar os dados da execução gerados por run_lab4.sh). 
Explique a variação dos resultados observados para os dois modos de execução.






