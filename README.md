# cloud-native-challenges
Challenges I worked on from various sources.


## BCFM

### TASK 0

### TASK 1

### TASK 2

### TASK 3
Create an app which serves "/bcfm" page. 
When a request sent it should have your name.

STEPS:

    cd task3/dockerapp
    docker build -t node-app:0.1 .
    docker images
    sudo docker run -p 4000:80 --name my-app node-app:0.1
    curl http://localhost:4000/bcfm

### TASK 4

### TASK 5

### TASK 6