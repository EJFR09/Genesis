# Primer Parcial
## 1. Procesamiento de Lenguajes
* Procesador de lenguaje
* Compilador 
	* Analisis
		* Lexico
		* Sintactico
		* Semantico 
	* Sintesis
		* Optimización
		* Construcción 
* Traductor
	* TDS
* Interprete
## 2. Gramaticas y BNF 
* Gramatica 
* BNF
	* Derivaciones
	* Arbol sintactico
	* Producciones
	* Alfabeto
	* Lenguaje
* GFG
* Gramatica Ambigua
* Gramatica Predictiva 
	* Eliminar Recursión por la izquierda
	* Eliminar Factor comun por la izquierda
	* Eliminar Factor comun indirecto
		* Truco es expandir desde abajo hacia arriba, hasta poder eliminar con factor comun directo
		* Si el problema es con un vacio podemos eliminar el de más alta jerarquia que puede no afectar
	* Ver el conjunto primero de las producciones
## 3. TDS
* Reglas semanticas
* Variables semanticas
* Atributos sintetizados
* Flujo
	- Escribís la BNF.
	- Escribís las reglas semánticas.
	- Convertis a predictiva
	- Transformas las reglas semánticas
	- Elegís una entrada de prueba.
	- Construís el árbol.
	- Anotás los atributos en cada nodo.
	- Verificás si el valor final sale correctamente.
	- Recién después hacés el pseudocódigo.
- Transformación de reglas semanticas
- Reglas practicas para pasar de BNF a Codigo 
	- Las funciones se ejecutan en orden A -> YXZ
	* Si aparece un terminal se usa match(terminal)
	* Si hay varias opciones se usa if input match(), else if input match 
	* Atributo sintetizado retorna
	* Atributo heredado se pasa como parametro 
## 4. Compilador 
