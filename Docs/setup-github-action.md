Refer this  : https://dev.to/knowbee/how-to-setup-continuous-deployment-of-a-website-on-a-vps-using-github-actions-54im


Step 1 - Open your terminal add ssh into your VPS
ssh user@hostname
cd ~/.ssh

Step 2 - Generate an ssh key
ssh-keygen -t rsa -b 4096 -C "test@example.com" //use github signed in email here

Step 3: Press Enter repeatedly to set default name(Don't set a passphrase)
Run ls to see the generated files

Step 4 - Add a public key to authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  

Note: We’re using >> so that the id_rsa.pub contents are appended to the end of the contents in the authorized_keys file, rather than override the contents in the authorized_keys.
Step 5 - Create GitHub secrets
cat ~/.ssh/id_rsa


Head over to your GitHub repository you wish to configure,click on
Settings -> ( side bar ) Secrets and Variables -> Actions -> Repository secrets -> Add new


HOST: set your hostname or ip address of the VPS
USERNAME: set the username you use to SSH into your VPS.
SSHKEY: set the key to your copied contents from the command above.
PORT: set the key to 22




Come into Local git repository
Create  .github/workflows/deploy.yml file

Add below in the deploy.yml

name: Deploy

# on: [push] # Trigger the workflow on push or pull request events
on:
  push:
    branches:
      - main  # Only trigger when there's a push to the 'main' branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
      with:
        ref: main

    - name: Copy repository contents via SCP
      uses: appleboy/scp-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        port: ${{ secrets.PORT }}
        key: ${{ secrets.SSHKEY }}
        source: "."
        target: "/var/www/port-folio"

    - name: Execute remote build and restart
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        port: ${{ secrets.PORT }}
        key: ${{ secrets.SSHKEY }}
        script: |
          cd /var/www/port-folio
          export PATH="$HOME/.local/share/pnpm:$PATH"
          pnpm install --frozen-lockfile
          pnpm build
          pm2 restart port-folio || pm2 start pnpm --name "port-folio" -- start


Push something to the main branch.

Check the workflow working in github repo -> Actions
And then check if the changes pull correctly in the repo in the VPS
