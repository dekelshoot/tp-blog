# creer un contenaire

## configuration du nouveau contenaire

    FROM node
    WORKDIR /app
    COPY . /app/
    RUN npm i
    EXPOSE 80
    CMD [ "npm", "start" ]

## creer un nouvelle image

    sudo docker build -t appimage .

## lancer une image

    sudo docker run -p 80:80 appimage

# docker compose

    docker-compose up
