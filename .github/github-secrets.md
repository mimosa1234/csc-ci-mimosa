name: Deploy HTML

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy index.html to CSC Pouta
        env:
          SSH_KEY: ${{ secrets.DEPLOY_SSH_KEY }}
          VM_IP: YOUR_VM_IP
          VM_USER: ubuntu
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H "$VM_IP" >> ~/.ssh/known_hosts

          # Kopioidaan ensin paikkaan johon user saa kirjoittaa
          scp -i ~/.ssh/deploy_key ./index.html "$VM_USER@$VM_IP:/tmp/index.html"

          # Siirretään sudo:lla Nginx webrootiin
          ssh -i ~/.ssh/deploy_key "$VM_USER@$VM_IP" \
            "sudo mv /tmp/index.html /var/www/html/index.html && sudo chown www-data:www-data /var/www/html/index.html"
