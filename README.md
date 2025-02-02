**Struction app**

![image](https://github.com/user-attachments/assets/3f9ca3c3-5859-4ad0-bba0-aff9a004f82f)

_**link crud flask app**_

https://github.com/buildwithdan/flask-crud

**Write Dockerfile**

    # Use an official Python runtime as a parent image
    FROM python:3.11-slim

    # Set the working directory to /app
    WORKDIR /app

    # Copy the current directory contents into the container at /app
    COPY . /app

    # Install pipenv
    RUN pip install pipenv

    # Install any needed packages specified in Pipfile
    RUN pipenv install
    # --deploy --ignore-pipfile

    EXPOSE 5000 
    # the port here doesnt mean anything.

    # Run flask when the container launches
    CMD ["pipenv", "run", "flask", "--app", "api/app.py", "--debug", "run", "--host=0.0.0.0", "--port=5000"]

**Write .env**

    DB_HOST=${DB_HOSTS} 
    DB_USERNAME=${DB_USERNAMES}
    DB_NAME=${DB_NAMES}
    DB_PASSWORD=${DB_PASSWORDS}
    DB_SCHEMA=${DB_SCHEMAS}
    DB_PORT=${DB_PORTS}

**Login DockerHub**
    
    docker login --username asdxxyy

**Create docker image and push DockerHub**

    docker build -t asdxxyy/py-flask .

    docker pull asdxxyy/py-flask

**Create Secret in Kubernetes**

    kubectl create secret generic mysec --from-literal DB_PASSWORDS=Abs123 --from-literal DB_USERNAMES=ulugbek --from-literal DB_NAMES=dbpy --from-literal DB_SCHEMAS=project_flask_crud --from-literal DB_HOSTS=db --from-literal DB_PORTS=5432

**Create Deployment in Kubernetes**

     apiVersion: apps/v1
     kind: Deployment
     metadata:
       creationTimestamp: null
       labels:
         app: crud
       name: crud
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: crud
       strategy: {}
       template:
         metadata:
           creationTimestamp: null
           labels:
             app: crud
         spec:
           containers:
           - image: ulugbekit94/py9
             name: py-crud
             envFrom:
             - secretRef:
                 name: mysec
             ports:
             - containerPort: 5000
             resources: {}
     status: {}

------------------------------------------------------------------------------------------------------------------------------------------    

**Via Helm**

   cd helm/crud
   
   helm create crud

**deployment.yaml**

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Release.Name }}-app
      labels:
        app: {{ .Release.Name }}-app
        chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
        release: {{ .Release.Name }}
        heritage: {{ .Release.Service }}
    spec:
      replicas: {{ .Values.replicaCount }}
      revisionHistoryLimit: {{ .Values.revisionHistoryLimit }}
      selector:
        matchLabels:
          app: {{ .Release.Name }}-app
      template:
        metadata:
          labels:
            app: {{ .Release.Name }}-app
        spec:
          containers:
          - name: {{ .Release.Name }}-app
            image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
            imagePullPolicy: {{ .Values.image.pullPolicy }}
            envFrom:
             - secretRef:
                 name: {{ .Values.secretName }} 
            ports:
              - containerPort: {{ .Values.containerPort }}
            resources:
              {{- toYaml .Values.resources | nindent 10 }}


**service.yaml**

    apiVersion: v1
    kind: Service
    metadata:
      name: {{ .Release.Name }}-srv
      labels:
        app: {{ .Release.Name }}-srv
        chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
        release: {{ .Release.Name }}
        heritage: {{ .Release.Service }}
    spec:
      ports:
      - port: {{ .Values.servicePort }}
        protocol: TCP
        targetPort: {{ .Values.containerPort }}
      selector:
        app: {{ .Release.Name }}-app
      type: {{ .Values.serviceType }}
    status:
      loadBalancer: {}


  **ingress.yaml**

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: {{ .Release.Name }}-ingress
      creationTimestamp: null
    spec:
      rules:
      - host: {{ .Values.domen }}
        http:
          paths:
          - backend:
              service:
                name: {{ .Release.Name }}-srv
                port:
                  number: {{ .Values.servicePort }}
            path: /
            pathType: {{ .Values.pathType }}
    status:
      loadBalancer: {}

**values.yaml**

    #Replicas
    replicaCount: 1
    revisionHistoryLimit: 1

    #Image
    image:
      repository: ulugbekit94/py
      tag: latest
      pullPolicy: IfNotPresent
      pullSecretName: my-pull-secret

    #Ports
    containerPort: 5000

    #Secrets
    secretName: mysec
    

    #Service-Params
    servicePort: 80
    #targetPort: 80
    serviceType: ClusterIP

    #Ingress-Params
    domen: kubs.uz
    pathType: Prefix


**commond**

    helm template crud ./crud/ --set "application.reloader=true" --debug > py-app.yaml

    helm install crud ./crud/
