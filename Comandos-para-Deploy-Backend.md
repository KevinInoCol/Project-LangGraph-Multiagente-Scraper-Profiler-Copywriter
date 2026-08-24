## Paso 0: Vinculación
gcloud init

## Paso 1: Creación del repositorio
gcloud artifacts repositories create repositorio-backend-langgraph-multiagente --repository-format docker --project datapath-ai17-kevin-inofuente --location us-central1

## Paso 2: Crear la imagen de mi APLICACION y subir al repositorio
gcloud builds submit --config=cloudbuild.yaml --project datapath-ai17-kevin-inofuente

    ## Si da error:
    gcloud projects add-iam-policy-binding datapath-ai17-kevin-inofuente \
        --member="user:kevin.inofuente.colque.27@gmail.com" \
        --role="roles/cloudbuild.builds.editor"

    gcloud services enable secretmanager.googleapis.com --project datapath-ai17-kevin-inofuente

## Paso 3: Comando para despliegue o ejecución de la imagen en el repositorio
gcloud run services replace service.yaml --region us-central1 --project datapath-ai17-kevin-inofuente

## Paso 4: OPCIONAL, Dar permisos de acceso a mi APLICACION. ESTO SE EJECUTA UNA SOLA VEZ
gcloud run services set-iam-policy servicio-backend-langgraph-multiagente-nombre-apellido gcr-service-policy.yaml --region us-central1 --project datapath-ai17-kevin-inofuente