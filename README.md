# Codigo-Final-Para-Circuito-Automatizado-de-Audiometria
https://arthur571souza.github.io/Codigo-Final-Para-Circuito-Automatizado-de-Audiometria/


//Bibliotecas utilizadas


#include <Keypad.h>

#include <LiquidCrystal.h>		

#include <Adafruit_NeoPixel.h>


///Constantes de modos de funcionamento

//modo de teste, funções mais limitadas

#define TESTE 2	

//modo real, funções completas

#define REAL 6	


///Constantes do teclado

//quantidade de linhas

#define QTDE_L 2		

//quantidade de colunas

#define QTDE_C 3		


///Constantes das fitas de led

//tamanho das fitas de led

#define QTDE_LEDs 12	

//pino da fita de led que representa o ouvido esquerdo

#define BLUE 6			

//pino da fita de led que representa o ouvido direito

#define RED A3			


///Constantes dos piezos

//pino que controla o volume do piezo direito

#define VOL_DIR 9		

//pino que controla o volume do piezo esquerdo

#define VOL_ESQ 10		

//pino que controla a frequencia do piezo direito

#define FREQ_DIR 7		

//pino que controla a frequencia do piezo esquerdo

#define FREQ_ESQ 8		


///Inicialização das fitas de LEDs

//Fita de led do ouvido direito(cor vermelha)

Adafruit_NeoPixel OUV_DIR = Adafruit_NeoPixel(QTDE_LEDs, RED, NEO_GRB + NEO_KHZ800);

//Fita de led do ouvido esquerda(cor azul)

Adafruit_NeoPixel OUV_ESQ = Adafruit_NeoPixel(QTDE_LEDs, BLUE, NEO_GRB + NEO_KHZ800);


///Simbolos para o LCD

byte Mao_DIR[8]={

				B00000,
				B00100,
				B00100,
				B10111,
				B01111,
				B01111,
				B00000
				
};

byte Mao_ESQ[8]={

				B00000,
				B00100,
				B00100,
				B11101,
				B11110,
				B11110,
				B00000
				
};

byte Ambos[8]={	
				
				B01000,
				B11111,
				B01000,
				B00000,
				B00010,
				B11111,
				B00010
				
};

byte Nenhum[8]={

				B00000,
				B10001,
				B01010,
				B00100,
				B01010,
				B10001,
				B00000
				
};

byte Confirmar[8]={
				
				B00000,
				B01000,
				B01000,
				B11111,
				B11111,
				B11111,
				B00000
};

byte Corrige[8]={
				
				B00000,
				B11111,
				B11111,
				B11111,
				B01000,
				B01000,
				B00000
};

byte cim_bai[8]={

				B00100,
				B01010,
				B10001,
				B00000,
				B10001,
				B01010,
				B00100
};

///Variavel para definir a configuração do teste(TESTE para modo teste e REAL para modo real)

int CONFIG_ATUAL=TESTE;


///Protótipos das funções

void emissao_som(int PIN_VOL, int PIN_FREQ, int* VOL, int FREQ);

void emissao_som(int* VOL_1, int* VOL_2, int FREQ);

int aquisicao_resp(void);

int pow_int(int base, int exp);

void impressao(char L1[], char L2[]);

void impressao(char L1[], byte L2,int x);


///Inicialização do teclado 2x3

/*
*							 Legenda                     
							 

1= Não ouviu				 
							 
10= Esquerdo				 

19= Ouviu os dois			 

C= Confirma				 
							 
R= Retorna					 							 

9= Direita					 
*/

char map_tecl[QTDE_L][QTDE_C]={

{1,10,19},

{'C','R',9}

};

byte pin_L[QTDE_L]={12, 11};

byte pin_C[QTDE_C]={A0, A1, A2};

Keypad teclado=Keypad(makeKeymap(map_tecl), pin_L, pin_C, QTDE_L, QTDE_C);

///Inicialização do lcd 16x2

LiquidCrystal lcd(5,4,3,2,1,0);


///Função para produzir som em apenas um piezo
void emissao_som(int PIN_VOL, int PIN_FREQ, int* VOL, int FREQ)
{

	//Variaveis para guardar as respostas do paciente
	char Resp_pac;
	
	//Variavel para loop
	int i;

	//Mensagem de instrução
	lcd.clear();
	lcd.setCursor(0,0);
	lcd.write("Atencao para");
	lcd.setCursor(0,1);
	lcd.write("o som");

	//Definição do volume
	analogWrite(PIN_VOL,*VOL*21);
	delay(500);
  
	//Loop para produzir pulsos de som
	for(i=0;i<(CONFIG_ATUAL/2);i++)
	{
		tone(PIN_FREQ,FREQ,(CONFIG_ATUAL*100));
		delay((CONFIG_ATUAL*100));
	}
	analogWrite(PIN_VOL,0);
	delay(500);
	
	//Função para receber resposta do paciente//
	Resp_pac=aquisicao_resp();

	//Analise da resposta, se ouvir diminui o volume, se não aumenta
	if(Resp_pac==PIN_VOL)
		*VOL=*VOL-1;
	else
		*VOL=*VOL+1;
		
}


///Função para produzir som nos dois piezos
void emissao_som(int* VOL_E, int* VOL_D, int FREQ)
{

	//Variaveis para guardar as respostas do paciente
	char Resp_pac;

	//Variavel para loop
	int i;

	//Mensagem de instrução
	lcd.clear();
	lcd.setCursor(0,0);
	lcd.write("Atencao para");
	lcd.setCursor(0,1);
	lcd.write("o som");

	//Definição do volume dos dois piezos
	analogWrite(VOL_ESQ,*VOL_E*21);
	analogWrite(VOL_DIR,*VOL_D*21);
	delay(500);

	//Loop para produzir pulsos de som alternantes entre piezos
	for(i=0;i<CONFIG_ATUAL;i++)
	{
		if(i%2==0)
			tone(FREQ_ESQ,FREQ,(CONFIG_ATUAL*100)/2);
		else
			tone(FREQ_DIR,FREQ,(CONFIG_ATUAL*100)/2);
		delay(((CONFIG_ATUAL*100)+100)/2);
	}
	analogWrite(VOL_ESQ,0);
	analogWrite(VOL_DIR,0);
	delay(500);
  
	//Função para receber resposta do paciente
	Resp_pac=aquisicao_resp();

	//Analise da resposta, como o som foi produzido nos dois lados
	//Se o paciente responder corretamente os dois volumes são diminuidos
	//Se ele indicar apenas um ouvido, esse tem o volume abaixado enquanto o outro aumenta
	if(Resp_pac==19)
	{
		*VOL_E=*VOL_E-1;
		*VOL_D=*VOL_D-1;
	}
	else if(Resp_pac==10)
	{
		*VOL_E=*VOL_E-1;
		*VOL_D=*VOL_D+1;
	}
	else if(Resp_pac==9)
	{
		*VOL_E=*VOL_E+1;
		*VOL_D=*VOL_D-1;
	}
	else
	{
		*VOL_E=*VOL_E+1;
		*VOL_D=*VOL_D+1;
	}
	
}


///Função para receber a resposta do usuario sobre o som produzido
int aquisicao_resp(void)
{

	//Variaveis para guardar as respostas do paciente
	char Resp_pac, Confir;
  
	//Loop para adquirir a resposta do paciente
	do{	

		//Impressão das instruções
		lcd.clear();
		lcd.setCursor(0,0);
		lcd.write("Aperte onde");
		lcd.setCursor(0,1);
		lcd.write("escutou o som");

		//Loop que apenas aceitas os botões direcionais
		do{
			Resp_pac=teclado.waitForKey();
		}while(Resp_pac!=9&&Resp_pac!=10&&Resp_pac!=1&&Resp_pac!=19);
		lcd.setCursor(0,0);
		lcd.clear();

		//Mensagens dependentes das escolhas
		switch(Resp_pac)
		{
			case 9:
			lcd.write("Confirmar DIR");
			lcd.setCursor(0,1);
			lcd.write("Corrigir escolha");
			break;
			case 10:
			lcd.write("Confirmar ESQ");
			lcd.setCursor(0,1);
			lcd.write("Corrigir escolha");
			break;
			case 19:
			lcd.write("Confirmar ambos");
			lcd.setCursor(0,1);
			lcd.write("Corrigir escolha");
			break;
			case 1:
			lcd.write("Confirmar nenhum");
			lcd.setCursor(0,1);
			lcd.write("Corrigir escolha");
			break;
		}

		//Loop que apenas aceita os botões "confirma" ou "corrige"
		do{
			Confir=teclado.waitForKey();
		}while(Confir!='C'&&Confir!='R');
	}while(Confir=='R');
	return Resp_pac;
	
}


///função para calculo de potência
int pow_int(int base, int exp)
{

	//Variavel para fazer o calculo
	int Resul=1;

	//Variavel para fazer o loop
	int i;
	
	//Loop para calcular a potencia
	for(i=0;i<exp;i++)
		Resul*=base;
	return Resul;
	
}


///Função para simplificar a impressão de textos nas duas linhas do LCD
void impressao(char L1[], char L2[])
{

	lcd.clear();
	lcd.setCursor(0,0);
	lcd.write(L1);
	lcd.setCursor(0,1);
	lcd.write(L2);
	delay(2500);
	
}


///Função para simplificar a impressão de palabras e sinais em qualquer linha
void impressao(char L1[], byte L2,int x)
{

	lcd.setCursor(0,x);
	lcd.write(L1);
	lcd.setCursor(15,x);
	lcd.write(L2);
	
}


void setup()
{	

	//Ativamento das fitas de led e do lcd
	OUV_DIR.begin();
	OUV_ESQ.begin();
	lcd.begin(16, 2);

	//Inicialização da aleatoriedade
	randomSeed(2);

	//Inicialização dos simbolos
	lcd.createChar(0,Mao_DIR);
	lcd.createChar(1,Mao_ESQ);
	lcd.createChar(2,Ambos);
	lcd.createChar(3,Nenhum);
	lcd.createChar(4,Confirmar);
	lcd.createChar(5,Corrige);
	lcd.createChar(6,cim_bai);
}


void loop()
{	

	//Mensagens introdutorias
	impressao("Oi! Bem-vindo","a audiometria");
	impressao("Atencao para","as intrucoes");
	impressao("Leia os nomes","dos botoes");
	lcd.clear();
	impressao("Direita",byte(0),0);
	impressao("Esquerda",byte(1),1);
	delay(2500);
	lcd.clear();
	impressao("Ambos",byte(2),0);
	impressao("Nenhum",byte(3),1);
	delay(2500);
	lcd.clear();
	impressao("Confirmar",byte(4),0);
	impressao("Corrigir",byte(5),1);
	delay(2500);
	impressao("Os botoes irao","funcionar no");
	impressao("momento correto.","Sera informado");
	impressao("o momento por","aqui");
	impressao("Qualquer duvida","tire com o");
	impressao("profissional","ao seu lado.");
	impressao("Bom exame"," ");

	//Variavel para guardar limiares por frequencia e a media tonal do ouvido direito
	int Lim_dir[CONFIG_ATUAL], Med_dir=0;

	//Variavel para guardar limiares por frequencia e a media tonal do ouvido esquerdo
	int Lim_esq[CONFIG_ATUAL], Med_esq=0;

	//Variavel para comparar a resposta atual com a anterior
	int Lim_mem_dir, Lim_mem_esq;	

	//Variavel para salvar faixa de idade
	int Faixa_idade=0;

	//Variaveis para bloqueio após aferição completa de cada ouvido
	bool Block_dir, Block_esq;

	//Ponteiros
	int *ptr_Lim_dir;
	int *ptr_Lim_esq;
	ptr_Lim_dir=&Lim_dir[0];
	ptr_Lim_esq=&Lim_esq[0];

  	//Variavel para guardar a resposta do paciente
	char Resp_pac;

	//Variavel para loop(talvez, vamos ver)
	int i,j;

	//Loop para inicializar vetor
	for(i=0;i<CONFIG_ATUAL;i++){
		Lim_dir[i]=6;
		Lim_esq[i]=6;
	}

	//Loop para receber a faixa de idade
	do{
		switch(Faixa_idade)
		{
			case 0:
			lcd.setCursor(0,0);
			lcd.print("informe idade");
			lcd.setCursor(13,0);
			lcd.print("<");
			lcd.setCursor(14,0);
			lcd.write(byte(6));
			lcd.setCursor(15,0);
			lcd.print(">");
			lcd.setCursor(0,1);
			lcd.print("Ate 7 anos");
			break;
			case 1:
			lcd.setCursor(0,0);
			lcd.print("informe idade");
			lcd.setCursor(13,0);
			lcd.print("<");
			lcd.setCursor(14,0);
			lcd.write(byte(6));
			lcd.setCursor(15,0);
			lcd.print(">");
			lcd.setCursor(0,1);
			lcd.print("Maior que 7 anos");
			break;
		}
		Resp_pac=teclado.waitForKey();
		if((Resp_pac==9||Resp_pac==1)&&Faixa_idade<1)
			Faixa_idade++;
		else if((Resp_pac==10||Resp_pac==19)&&Faixa_idade>0)
			Faixa_idade--;
		lcd.clear();
	}while(Resp_pac!='C');

	//Loop para percorrer cada posição dos vetores.
	//Cada posição irá salvar o limiar nas seguintes frequencias
	//250hz, 500hz, 1000hz, 2000hz, 4000hz e 8000hz
	//(para testes o loop tem de ser de 0-1, na realidade vai de 0-5)
	for(i=0;i<CONFIG_ATUAL;i++)
	{
		//Reinicialização das variaveis
		Block_dir=true;
		Block_esq=true;
		Lim_mem_dir=6;
		Lim_mem_esq=6;
		j=0;

		//Loop para alternar a fonte das orelhas(provavelmente aleatoriezar depois usando função random)
		do{
			j=random(25);

			//Se o limiar de um ouvido não foi descoberto ainda irá entrar nos ifs e mandar os dados para
			//função 'emissao_som' variando o volume a cada iteração
			if(j==19&&Block_dir&&Block_esq)
				emissao_som(ptr_Lim_esq+i,ptr_Lim_dir+i,(250*pow_int(2,i)));
			else if(j%2==0&&Block_dir)
				emissao_som(VOL_DIR,FREQ_DIR,ptr_Lim_dir+i,(250*pow_int(2,i)));
			else if(j%2==1&&Block_esq)
				emissao_som(VOL_ESQ,FREQ_ESQ,ptr_Lim_esq+i,(250*pow_int(2,i)));
			
			//Os ifs abaixo verificam se o paciente escutou o primeiro som no ouvido direito. 
			//Se não escutou, ele verifica nas proximas iterações o limite que o paciente escuta
			if((Lim_dir[i]>6||Lim_mem_dir>6))
			{
				if(Lim_dir[i]<Lim_mem_dir)
				{
					Lim_dir[i]=Lim_mem_dir;
					Block_dir=false;
				}
				else if(Lim_dir[i]>Lim_mem_dir)
					Lim_mem_dir=Lim_dir[i];
				else if(Lim_dir[i]==13)
					Block_dir=false;
			}
			
			//Os ifs abaixo verificam se o paciente escutou o primeiro som no ouvido esquerdo. 
			//Se escutou, ele verifica nas proximas iterações o limite que o paciente não escuta
			else if((Lim_dir[i]<6||Lim_mem_dir<6))
			{
				if(Lim_dir[i]>Lim_mem_dir)
					Block_dir=false;
				else if(Lim_dir[i]<Lim_mem_dir)
					Lim_mem_dir=Lim_dir[i];
				else if(Lim_dir[i]==0)
					Block_dir=false;
			}
			
			//Os ifs abaixo verificam se o paciente escutou o primeiro som no ouvido esquerdo.
			//Se não escutou, ele verifica nas proximas iterações o limite que o paciente escuta
			if((Lim_esq[i]>6||Lim_mem_esq>6))
			{
				if(Lim_esq[i]<Lim_mem_esq)
				{
					Lim_esq[i]=Lim_mem_esq;
					Block_esq=false;
				}
				else if(Lim_esq[i]>Lim_mem_esq)
					Lim_mem_esq=Lim_esq[i];
				else if(Lim_esq[i]==13)
					Block_esq=false;
			}
			
			//Os ifs abaixo verificam se o paciente escutou o primeiro som. 
			//Se escutou, ele verifica nas proximas iterações o limite que o paciente não escuta
			else if((Lim_esq[i]<6||Lim_mem_esq<6))
			{
				if(Lim_esq[i]>Lim_mem_esq)
					Block_esq=false;
				else if(Lim_esq[i]<Lim_mem_esq)
					Lim_mem_esq=Lim_esq[i];
				else if(Lim_esq[i]==0)
					Block_esq=false;
			}
		}while(Block_dir||Block_esq);
	}

	//Calculo da media
	lcd.clear();
	for(i=0;i<CONFIG_ATUAL;i++){
		Med_dir=Med_dir+(Lim_dir[i]*10);
		Med_esq=Med_esq+(Lim_esq[i]*10);
	}
	Med_dir=Med_dir/CONFIG_ATUAL;
	Med_esq/=CONFIG_ATUAL;
	
	//Loop para apresentar os resultados
	do{
		//Reinicialização da variavel j, do lcd e das fitas de led
		j=0;
		lcd.clear();
		for(i=11;i>=0;i--)
			OUV_DIR.setPixelColor(i, OUV_DIR.Color(0, 0, 0));
		for(i=11;i>=0;i--)
			OUV_ESQ.setPixelColor(i, OUV_ESQ.Color(0, 0, 0));
		OUV_DIR.show();
		OUV_ESQ.show();
		
		//Loop para mostrar qual resultado vai ser apresentado ou se vai desligar o aparelho
		do{
			switch(j)
			{
				case 0:
				lcd.setCursor(0,0);
				lcd.print("Resultado");
				lcd.setCursor(13,0);
				lcd.print("<");
				lcd.setCursor(14,0);
				lcd.write(byte(6));
				lcd.setCursor(15,0);
				lcd.print(">");
				lcd.setCursor(0,1);
				lcd.print("Media tonal");
				break;
				case 1:
				lcd.setCursor(0,0);
				lcd.print("Resultado");
				lcd.setCursor(13,0);
				lcd.print("<");
				lcd.setCursor(14,0);
				lcd.write(byte(6));
				lcd.setCursor(15,0);
				lcd.print(">");
				lcd.setCursor(0,1);
				lcd.print("Limiar p/ freq");
				break;
				case 2:
				lcd.setCursor(0,0);
				lcd.print("Resultado");
				lcd.setCursor(13,0);
				lcd.print("<");
				lcd.setCursor(14,0);
				lcd.write(byte(6));
				lcd.setCursor(15,0);
				lcd.print(">");
				lcd.setCursor(0,1);
				lcd.print("Desligar");
				break;
			}
			Resp_pac=teclado.waitForKey();
			if((Resp_pac==9||Resp_pac==1)&&j<2)
				j++;
			else if((Resp_pac==10||Resp_pac==19)&&j>0)
				j--;
			lcd.clear();
		}while(Resp_pac!='C');
		
		//Mostra a media tonal de cada ouvido alem do estado dele segundo a escala de Lloyd e Kaplan
		if(j==0)
		{
			do{
				lcd.clear();
				lcd.setCursor(0,0);
				lcd.print("E:");
				if(Med_esq==0||Med_esq==13)
					lcd.print("S.E");
				else
					lcd.print(Med_esq);
				lcd.setCursor(5,0);
				lcd.print("dB");
				lcd.setCursor(9,0);
				lcd.print("D:");
				if(Med_dir==0||Med_dir==13)
					lcd.print("S.E");
				else
					lcd.print(Med_dir);
				lcd.setCursor(14,0);
				lcd.print("dB");
				lcd.setCursor(0,1);
				
				//Escala de Lloyd e Kaplan para pessoas maiores que 7 anos
				if(Faixa_idade==1)
				{
					if(Med_esq<=25)
					{
						lcd.print("Normal");
					}
					else if(Med_esq<=40)
					{
						lcd.print("P. Leve");
					}
					else if(Med_esq<=55)
					{
						lcd.print("P. Mod.");
					}
					else if(Med_esq<=70)
					{
						lcd.print("P. MSe.");
					}
					else if(Med_esq<=90)
					{
						lcd.print("P. Sev.");
					}
					else
					{
						lcd.print("p. Pro.");
					}
					lcd.setCursor(9,1);
					if(Med_dir<=25)
					{
						lcd.print("Normal");
					}
					else if(Med_dir<=40)
					{
						lcd.print("P. Leve");
					}
					else if(Med_dir<=55)
					{
						lcd.print("P. Mod.");
					}
					else if(Med_dir<=70)
					{
						lcd.print("P. MSe.");
					}
					else if(Med_dir<=90)
					{
						lcd.print("P. Sev.");
					}
					else
					{
						lcd.print("p. Pro.");
					}
				}

				//Escala de Northern e Down para pessoas menores que 7 anos
				else					
				{
					if(Med_esq<=15)
					{
						lcd.print("Normal");
					}
					else if(Med_esq<=25)
					{
						lcd.print("P. Dis.");
					}
					else if(Med_esq<=40)
					{
						lcd.print("P. Leve");
					}
					else if(Med_esq<=65)
					{
						lcd.print("P. Mod.");
					}
					else if(Med_esq<=95)
					{
						lcd.print("P. Sev.");
					}
					else
					{
						lcd.print("P. Pro.");
					}
					lcd.setCursor(9,1);
					if(Med_dir<=15)
					{
						lcd.print("Normal");
					}
					else if(Med_dir<=25)
					{
						lcd.print("P. Dis.");
					}
					else if(Med_dir<=40)
					{
						lcd.print("P. Leve");
					}
					else if(Med_dir<=65)
					{
						lcd.print("P. Mod.");
					}
					else if(Med_dir<=95)
					{
						lcd.print("P. Sev.");
					}
					else
					{
						lcd.print("P. Pro.");
					}
				}
				for(i=0;i<(Med_esq/10);i++)
					OUV_ESQ.setPixelColor(i, OUV_ESQ.Color(0, 0, 255));
				for(i=11;i>=Med_esq/10;i--)
					OUV_ESQ.setPixelColor(i, OUV_ESQ.Color(0, 0, 0));
				for(i=0;i<Med_dir/10;i++)
					OUV_DIR.setPixelColor(i, OUV_DIR.Color(255, 0, 0));
				for(i=11;i>=Med_dir/10;i--)
					OUV_DIR.setPixelColor(i, OUV_DIR.Color(0, 0, 0));
				OUV_DIR.show();
				OUV_ESQ.show();
				Resp_pac=teclado.waitForKey();
			}while(Resp_pac!='R');
		}

		//Mostra os limiares por frequencia de cada ouvido
		else if(j==1)
		{
			j=0;

			//Loop para escolher qual frequencia mostrar na escala de Leds
			do{
				lcd.clear();
				lcd.setCursor(0,0);
				lcd.print("F:");
				lcd.print((250*pow_int(2,j)));
				lcd.print("hz");
				lcd.setCursor(9,0);
				lcd.print("E:");
				if(Lim_esq[j]==0||Lim_esq[j]==13)
					lcd.print("S.E");
				else
					lcd.print(Lim_esq[j]*10);
				lcd.setCursor(14,0);
				lcd.print("dB");
				lcd.setCursor(9,1);
				lcd.print("D:");
				if(Lim_dir[j]==0||Lim_dir[j]==13)
					lcd.print("S.E");
				else
					lcd.print(Lim_dir[j]*10);
				lcd.setCursor(14,1);
				lcd.print("dB");
				for(i=0;i<Lim_esq[j];i++)
					OUV_ESQ.setPixelColor(i, OUV_ESQ.Color(0, 0, 255));
				for(i=11;i>=Lim_esq[j];i--)
					OUV_ESQ.setPixelColor(i, OUV_ESQ.Color(0, 0, 0));
				for(i=0;i<Lim_dir[j];i++)
					OUV_DIR.setPixelColor(i, OUV_DIR.Color(255, 0, 0));
				for(i=11;i>=Lim_dir[j];i--)
					OUV_DIR.setPixelColor(i, OUV_DIR.Color(0, 0, 0));
				OUV_DIR.show();
				OUV_ESQ.show();
				Resp_pac=teclado.waitForKey();
				if((Resp_pac==9||Resp_pac==1)&&j<(CONFIG_ATUAL-1))
					j++;
				else if((Resp_pac==10||Resp_pac==19)&&j>0)
					j--;
			}while(Resp_pac!='R');
			j=0;
		}
	}while(j!=2);
	
}
