# Rasterização de primitivas

## Atividade

Esta atividade foi realizada para a disciplina de Introdução à Computação Gráfica, proposta no intuito de familiarizar os alunos com os algoritmos de rasterização de primitivas utilizados em computação gráfica. O trabalho foi desenvolvido sobre um framework que simula acesso direto à memória de vídeo, implementado com OpenGL, feito e disponibilizado pelo professor. O objetivo é implementar algoritmos para a rasterização de pontos e linhas, sendo que a cor de cada linha deve ser obtida através da interpolação linear das cores de seu vértice inicial e final.

## Implementação

O padrão de cores RGBA é usado em cada pixel e seus canais são representados por 1 byte cada.

#### Trecho de código:

~~~c
	typedef struct{

    		unsigned char R, G, B, A;

	} tRGBA;

	typedef struct{

    		int x, y; // Coordenadas na janela
    		tRGBA cor;

	} tPixel;
~~~	

O algoritmo de Bresenham foi generalizado e utilizado para geração de linhas nos oito octantes do sistema cartesiano.

### PutPixel()

Tendo acesso direto à memória, concedido pelo framework através do ponteiro FBptr que simula a escrita no Color Buffer, cada pixel pode ser aceso na parte da tela especificada pelas suas coordenadas X e Y e ter a intensidade de seus canais de cor alterados. Tomando W como a largura total da janela de exibição, pode-se achar a posição inicial de um pixel no Color Buffer (componente R) com o auxílio da fórmula:

![](https://github.com/Moura00010001/CG/blob/master/Atividade%201/Printscreens/Componente%20R.png)

#### Trecho de código:	

	// Posição do primeiro byte de cada pixel
    	componente = (pixel->x*4) + (pixel->y*IMAGE_WIDTH*4);

    	FBptr[componente] = pixel->cor.R; // Componente R do pixel
    	FBptr[++componente] = pixel->cor.G; // Componente G do pixel
    	FBptr[++componente] = pixel->cor.B; // Componente B do pixel
    	FBptr[++componente] = pixel->cor.A; // Componente A do pixel

####Printscreen: PutPixel() - Rasterização dos pontos

###DrawLine()
O algoritmo de rasterização de linhas proposto na atividade para ser implementado, como já foi citado anteriormente, foi o algoritmo de Bresenham. Considerado muito eficiente em razão de suas operações aritméticas com números inteiros, ele se limita a desenhar retas apenas no 1º octante, com ângulos entre 0 e 45°.

####Printscreen: DrawLine() - Bresenham puro

Entretanto, o algoritmo foi generalizado a partir dos casos que aparecem na tabela a seguir, que também contém os respectivos tratamentos necessários para a rasterização de retas com ângulos entre 45° e 360°. Essa generalização atém-se ao balanço do comportamento de dx e dy nos octantes.

####Tabela:		...Octantes:

Como consequência, pode-se rasterizar linhas em todas as direções do plano cartesiano.

####Printscreen: DrawLine() - 8 octantes

###InterpolaCor()
Na transição de cor entre os vértices de cada linha foi utilizado uma porcentagem que ajudará na informação da quantiade de cor que o novo pixel que é aceso durante as iterações do Bresenham deverá conter do pixel final. Essa porcentagem é calculada pela (distância do pixel atual que será aceso até o pixel final) dividido pela (distância do pixel inicial ao pixel final, que dá o comprimento da reta).

####Trecho de código:
	tPixel InterpolaCor(float p, tPixel* pixel1, tPixel* pixel2){

    		tPixel pixel;
    		pixel.cor.R = p*(pixel1->cor.R) + (1 - p)*(pixel2->cor.R);
    		pixel.cor.G = p*(pixel1->cor.G) + (1 - p)*(pixel2->cor.G);
    		pixel.cor.B = p*(pixel1->cor.B) + (1 - p)*(pixel2->cor.B);
    		pixel.cor.A = p*(pixel1->cor.A) + (1 - p)*(pixel2->cor.A);

    		return pixel;

	}

####Printscreen: InterpolaCor() - Interpolação de cores

###DrawTriangle()
Na rasterização do triângulo é utilizado as coordenadas dos pixels passados por parâmetros como os vértices deste polígono. DrawLine é chamada três vezes para formar suas arestas.

####Trecho de código:
	void DrawTriangle(tPixel* pixel1, tPixel* pixel2, tPixel* pixel3){

    		DrawLine(pixel1, pixel2);
    		DrawLine(pixel2, pixel3);
    		DrawLine(pixel3, pixel1);

	}

####Printscreen: DrawTriangle() - Arestas do triângulo

Para evitar problemas com erros de Falha de segmentação ou formas que parecem espelhadas devido ao acesso indefinido aos índices do array que simula a memória de vídeo, realiza-se uma checagem nos campos x e y para garantir que o acesso se restrinja as áreas que representam efetivamente os pixels desejados na janela de exibição.

####Trecho de código:
	if(pixel->x < 0 || pixel->y < 0 ||
       	   pixel->x > IMAGE_WIDTH || pixel->y > IMAGE_HEIGHT){

       		/* Tentativa inválida de acessar uma posição que não
        		consta nos limites do sistema de coordenadas da janela*/

       		return 1;

    	}

####Printscreen: Falha de segmentação - Sem checagem
####Printscreen: Falha de segmentação - Com checagem

###PreencheTriangulo()
Utilizando coordenadas baricêntricas podemos verificar se um ponto pertence ou não a um triângulo. "Parte-se do fato de que qualquer ponto P do triângulo pode ser definido a partir das coordenadas de seus três vértices, de modo que P = w1P1 + w2P2 + w3P3, onde w1, w2 e w3 são números reais e w1 + w2 + w3 = 1 . Os coeficientes w1, w2 e w3 são denominados coordenadas baricêntricas de P em relação a P1, P2 e P3." - MundoGEO.
O seguinte sistema de equações é gerado:

####Printscreen: Sistema de equações

Reorganizando várias vezes podemos resolver w1, w2 e w3 para qualquer ponto com as seguintes equações:

####Printscreen : w1, w2 e w3

Com as coordenadas do pixel a ser analisado e dos pixels que representam os vértices do triângulo, calcula-se as coordenadas baricêntricas. As três coordenadas devem ser positivas para o ponto pertencer ao interior do triângulo e se ao menos uma delas for negativa, o ponto estará fora dele.

####Trecho de código:
	if(w1 >= 0 && w2 >= 0 && (w1 + w2) <= 1){
		
		// Trecho de código da interpolação de cores
	
	}

####Printscreen: Coordenadas baricêntricas

A função PreencheTriangulo() percorre todos os pixels da janela de exibição e acende os que estão dentro do triângulo. A interpolação das cores é feita a partir da soma da multiplicação das coordenadas baricêntricas por cada componente da cor dos pixels que representam os vértices do triângulo.

####Trecho de código:
	cor = (w1*pixel1->cor.R + w2*pixel2->cor.R + w3*pixel3->cor.R) + 0.5;
                 pixel.cor.R = (char) cor;
                 cor = (w1*pixel1->cor.G + w2*pixel2->cor.G + w3*pixel3->cor.G) + 0.5;
                 pixel.cor.G = (char) cor;
                 cor = (w1*pixel1->cor.B + w2*pixel2->cor.B + w3*pixel3->cor.B) + 0.5;
                 pixel.cor.B = (char) cor;
                 cor = (w1*pixel1->cor.A + w2*pixel2->cor.A + w3*pixel3->cor.A) + 0.5;
                 pixel.cor.A = (char) cor;

####Printscreen: PreencheTriangulo() - Face do triângulo

##Resultados

As funções implementadas realizam bem a rasterização das primitivas gráficas propostas pela atividade mas poderiam ser mais eficientes com a possibilidade de evitar os cálculos da porcentagem e da distância entre o próximo pixel a ser aceso até o pixel final da reta utilizados na função InterpolaCor() e de outra abordagem no preenchimento do triângulo, sem coordenadas baricêntricas, talvez com uma variação do algoritmo de Bresenham desenhando linhas horizontais para preencher o triângulo.
