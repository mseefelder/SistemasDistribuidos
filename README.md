# Lista 2 - Sistemas Distribuídos

## Questão 1

Os dois principais modelos de Inter Process Communication (IPC) são os de memória compartilhada e troca de mensagens.

* Troca de mensagens: Nesse modelo, os processos trocam mensagens através de chamadas de sistema `send()` e `receive()` que põe e retiram mensagens de uma fila armazenada em *kernel space*. O enlace pode ser direto ou indireto e a troca de mensagens pode ser síncrona ou assíncrona:
	* Direto/Indireto:
		* No caso do enlace **direto**, tanto a **origem** quanto o **destino devem ser identificados** nas chamadas de `send` e `receive`. Nesse caso os enlaces são **unidirecionais** e **criados automaticamente** (realizado com o uso de **sinais** por exemplo);
		* No caso **indireto**, as mensagens sao enviadas para e lidas a partir de uma **"caixa postal"**. Nesse caso, as caixas postais devem ser **criadas explicitamente** e são **bidirecionais** (realizado com **sockets** ou **pipes**, por exemplo); 
	* Síncrono/Assíncrono:
		* No caso **síncrono**, as **chamadas são bloqueantes**, ou seja, o receptor espera a mensagem chegar e o emissor espera a mensagem ser recebida. Durante a espera, o processo fica em *Waiting*;
		* No caso **assíncrono**, as **chamadas são não-bloqueantes**, ou seja, após as chamadas os processos continuam executando normalmente. Nessa situação as mensagens são armazenadas em um *buffer* em *kernel space* de tamanho limitado, que pode ser controlado pelo processo.
As principais vantagens das trocas de mensagens é que o sistema operacional oferece as chamadas que facilitam sua realização e administra aspectos como os *buffers* da troca assíncrona, por exemplo. Uma desvantagem é que tudo é baseado em chamadas de sistema, que tomam bastante tempo de execução. 

* Memória compartilhada: Nesse modelo uma região de memória é mapeada para ser compartilhada entre dois processos. Diferente do caso de troca de mensagens, o sistema operacional não está envolvido nesse processo, e a região de memória compartilhada está **fora do *kernel space***. Isso acarreta numa desvantagem, que é a grande chance da ocorrência de *condições de corrida* no acesso dessa região, sendo necessária a **coordenação do acesso através de sincronização **.

## Questão 2

Em se tratando de pipes, uma escrita (`write()`) bloqueante bloqueia quando não há espaço no *buffer* do *pipe*, enquanto uma não-bloqueante simplesmente retorna um valor de erro e a execução continua. No caso da leitura (`read()`), uma leitura bloqueante bloqueia até que haja algo para ler, equanto a não-bloqueante retorna um valor de erro no caso de não haver o que ler e a execução continua.

No caso do código, temos as seguintes possíveis combinações:

* *Write* **bloqueante** & *Read* **bloqueante**: *P1* escreve continuamente no *pipe* e *P2* lê sempre que há algo para ser lido. Como a função `process()` pode demorar um pouco para retornar é improvável que não haja nada para ser lido por *P2*, mas pode ocorrer do *buffer* ficar cheio (por *P2* não dar conta da velocidade com que *P1* o preenche). Nesse caso, *P1* vai bloquear até que haja espaço para a escrita no *pipe*. A função `process()` sempre receberá algum parâmetro que *P1* enviou;
* *Write* **bloqueante** & *Read* **não-bloqueante**: Temos uma situação parecida com a anterior, pois como `process()` pode demorar para retornar, é improvável que tente ler o *buffer* e não haja nada para ser lido. Caso isso acontecesse, o valor **m** lido em `process()` provavelmente seria preenchido com zeros, não garantindo que *P2* recebesse sempre valores enviados por *P1*;
* *Write* **não-bloqueante** & *Read* **não-bloqueante**: Nesse caso pode ocorrer de *P1* tentar escrever com o *buffer* cheio e não bloquear. Numa situação dessas, a escrita não será feita e a execução de *P1* continuará normalmente. *P2* iria provavelmente receber apenas valores enviados por *P1*, mas não necessariamente receberia todos os valores que P1 tentou enviar. Com pouquíssima probabilidade (devido à demora de `process()`) poderia ocorrer de P2 ler e o *buffer*, acarretando numa situação descrita no tópico anterior;
* *Write* **não-bloqueante** & *Read* **bloqueante**: Nesse caso pode ocorrer de *P1* tentar escrever com o *buffer* cheio e não bloquear. Numa situação dessas, a escrita não será feita e a execução de *P1* continuará normalmente. *P2* irá receber sempre valores enviados por *P1*, mas não necessariamente receberia todos os valores que P1 tentou enviar.

## Questão 3

Cada processo é associado a um **PCB** (Process Control Block), grande estrutura de dados mantida para armazenar todo o estado do processo, o que é bem custoso para o Sistema Operacional. Além disso a troca de contexto entre processos é uma operação extremamente custosa para o Sistema Operacional. Um sistema *multi-process* envolve todos esses problemas além das diversas trocas de contexto que são necessárias para realizar a comunicação entre processos.

Por outro lado, num sistema *multi-threaded*, múltiplas threads compartilham um único **PCB**, por sua vez tendo um custo menor para o Sistema Operacional em questões de consumo de memória. Além disso, um mesmo processo pode aproveitar mais *cores* do processador e a administração de atividades concorrentes é facilitada. 

Sistemas de com *threads* podem usá-las em dois modelos: *User-Level Threads* e *Kernel-Level Threads*:
* **User-Level Threads** rodam no espaço do usuário, fazendo com que as operações de criar e destruir threads sejam meramente operações de alocação e liberação de memória além da troca de contexto de threads poder ser realizada de forma barata, com poucas operações (pois a quantidade de dados armazenados específios de cada *thread* e pequena). Uma desvantagem é que no caso de uma *thread* bloquear em alguma instrução, o processo todo bloqueia, pois o Sistema Operacional não tem conhecimento da existência de diversas threads dentro do processo e o enxerga como um todo para questões de escalonamento;
* **Kernel-Level Threads** (ou *Lightweght Processes*) são implementações de *threads* visíveis como unidades individuais para o Sistema Operacional, pois são implemetadas a nível de *kernel*, fazendo com que uma *thread* bloqueada não bloqueie seu processo inteiro. Por outro lado, todas as operações de administração das *threads* passam a ser chamadas de sistema, tornando o custo da mudança de contexto entre threads alto;
* Para obter o melhor das duas opções, pode-se utilizar um modelo híbrido de *threads* que mistura *kernal-level threads* com *user-level threads*.

Uma **possível desvantagem** de sistemas *multi-threaded* é que, como as implementações de *threads* aramazenam o mínimo possível de informação que permite a divisão da CPU entre diferentes *threads*, proteger o acesso inapropriado de dados pelas diversas *threads* é uma tarefa deixada para o programador.

## Questão 4

Um sistema *multi-threaded* baseado em *user-level threads* é um processo que só roda em um núcleo de processamento de cada vez, pois as *threads* em nivel de usuário não são enxergadas pelo Sistema Operacional como unidades que podem ser escalonadas independentemente. Portanto, é de se esperar que o desempenho de tal sistema sem um computador multi-processado seja igual ao de um mono-processado no sentido de que o mesmo não aproveita a disponibilidade de múltiplos *cores* para a execução concomitante de *threads*.

## Questão 5



## Questão 6

Pode ocorrer, por exemplo, um *deadlock* na seguinte situação:

1. A *thread T1* faz uma chamada `acquire()` e executa a linha `lock->interested[this_thread] = 1`;
2. Ocorre uma **troca de contexto** para a *thread T2*;
3. A *thread T2* faz uma chamada `acquire()` e executa a linha `lock->interested[this_thread] = 1`;

A partir de agora, **ambas as *threads* ficam presas no loop de `while`**, pois ambas as posições de `lock->interested[]` são verdadeiras (`== 1`).

## Questão 7

```
//Se lock==false, está livre
//Se lock==true, está trancada
//Começa livre
bool lock = false;

acquire(bool lock) {
	bool acquired = true;
	while(acquired) {
		swap(&acquired, &lock);
	}
}

release(bool lock) {
	bool temp = false;
	swap(&lock, &temp);
}
```

## Questão 8



## Questão 9



## Questão 10



## Questão 11



## Questão 12



## Questão 13



## Questão 14



## Questão 15



## Questão 16



## Questão 17



## Questão 18



## Questão 19



## Questão 20


