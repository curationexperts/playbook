# Docker for containers

## Over VPN or other private networks
- Sometimes the IP range that Docker automatically assigns your private network conflicts with either your home/office internet connection, or an institution's Virtual Private Network (VPN).
- If you cannot connect to an IP address from within your Docker container that you can connect to outside of that container:
  - Verify by using curl from the command line first on your local file system. If the site is using HTTP auth, you can pass that in as well
  ```bash
  curl --user YOUR_USERNAME:YOUR_PASSWORD https://some-url-behind-a-vpn.library.institution.edu  
  ```
  then bash into your docker container
  ```bash
  docker-compose exec YOUR_SERVICE_HERE bash
  ```
  and run the same command as before, this time from inside your container
  ```bash
  curl --user YOUR_USERNAME:YOUR_PASSWORD https://some-url-behind-a-vpn.library.institution.edu  
  ```
  - If the above works outside, but not inside, your container, then you can change the IP range that Docker assigns your local Docker network to (how the containers talk to each other on your machine).
  - On a Mac, you can do this by
    - Open Docker Desktop --> Preferences --> Docker Engine and replace the existing configuration with the one below
    ```
    {
      "debug": true,
      "default-address-pools": [
        {
          "base": "10.160.0.0/16",
          "size": 24
        }
      ],
      "experimental": false
    }
    ```
    then click on "Apply & Restart" in the bottom right corner
    - Once you've changed your Docker configuration, you will need to take down any running containers
    ```bash
    docker-compose down
    ```
    then remove the existing local Docker network
    ```bash
    docker network prune
    ```
    - At this point you can re-start your service 
    ```bash
    docker-compose up YOUR_SERVICE_HERE
    ```
    then bash into your docker container
    ```bash
    docker-compose exec YOUR_SERVICE_HERE bash
    ```
    and run the same command as before, this time from inside your container, now running on a freshly minted private network
    ```bash
    curl --user YOUR_USERNAME:YOUR_PASSWORD https://some-url-behind-a-vpn.library.institution.edu  
    ```
