
name: Instalar Servidor de Area de Trabalho

on:
  workflow_dispatch:

jobs:
  desktop:
    runs-on: ubuntu-22.04
    timeout-minutes: 360

    steps:
      - name: Instalar XFCE e VNC
        run: |
          sudo apt-get update
          sudo apt-get install -y \
            xfce4 xfce4-goodies \
            tigervnc-standalone-server \
            dbus-x11 wget unzip

      - name: Instalar Ngrok
        env:
          NGROK_AUTHTOKEN: ${{ secrets.NGROK_AUTHTOKEN }}
        run: |
          wget -q https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.zip
          unzip -o ngrok-v3-stable-linux-amd64.zip
          sudo install ngrok /usr/local/bin/ngrok
          ngrok config add-authtoken "$NGROK_AUTHTOKEN"

      - name: Configurar VNC
        env:
          VNC_PASSWORD: ${{ secrets.VNC_PASSWORD }}
        run: |
          mkdir -p ~/.vnc
          printf '%s\n' "$VNC_PASSWORD" |
            vncpasswd -f > ~/.vnc/passwd
          chmod 600 ~/.vnc/passwd

          cat > ~/.vnc/xstartup <<'EOF'
          #!/bin/sh
          unset SESSION_MANAGER
          unset DBUS_SESSION_BUS_ADDRESS
          exec startxfce4
          EOF

          chmod +x ~/.vnc/xstartup

      - name: Iniciar Desktop
        run: |
          vncserver :1 \
            -localhost yes \
            -geometry 1280x720 \
            -depth 24

          sleep 3
          echo "Servidor VNC iniciado na porta 5901."

      - name: Expor conexão pelo Ngrok
        run: |
          nohup ngrok tcp 5901 --log=stdout \
            > "$RUNNER_TEMP/ngrok.log" 2>&1 &

          sleep 5
          cat "$RUNNER_TEMP/ngrok.log"

          if ! grep -q 'url=tcp://' "$RUNNER_TEMP/ngrok.log"; then
            echo "Confira os logs do Ngrok para ver o erro."
            exit 1
          fi

          echo "Consulte os logs para obter o endereço TCP."
          sleep 21000
          
