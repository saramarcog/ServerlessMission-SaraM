# NOCTURNA · Senderismo Nocturno (Serverless AWS)

Proyecto para la misión serverless. Es una landing page para reservar plazas en rutas de un Club de senderismo nocturno, conectada a un backend serverless en AWS.

## Enlaces del proyecto
- **GitHub Pages:** https://saramarcog.github.io/ServerlessMission-SaraM/
- **Amazon S3:** http://nighthike-frontend-saram-2026.s3-website-us-east-1.amazonaws.com/
- **API Gateway (POST):** https://7jjt9avvlc.execute-api.us-east-1.amazonaws.com/dev/reservations

## Cómo funciona la arquitectura
1. El usuario entra a la web (alojada en S3 o GitHub Pages) y rellena el formulario con su nombre, email y la ruta elegida.
2. Al pulsar en enviar, el frontend hace una petición `POST` en formato JSON al endpoint de **API Gateway**.
3. API Gateway invoca a la función **AWS Lambda** (`nighthike-register-handler`, escrita en Node.js 20).
4. La Lambda valida los datos recibidos y:
   - Guarda la reserva en la tabla de **DynamoDB** (`nighthike-reservations`).
   - Envía un mensaje a un tema de **Amazon SNS** (`nighthike-confirmations-topic`), que notifica por correo la nueva reserva.
5. Si todo ha ido bien, la Lambda devuelve un código `200` con el identificador de la reserva y el frontend muestra el mensaje de confirmación en verde.

## Decisiones técnicas
- **Clave primaria (Partition Key):** En DynamoDB he usado un identificador único generado con UUID v4 (`reservationsId`). De esta forma evitamos usar el email como clave, lo que permite que una misma persona pueda hacer varias reservas diferentes sin que se sobreescriban los datos.
- **Seguridad e IAM:** La Lambda se ejecuta usando su propio Execution Role con los permisos necesarios para escribir en DynamoDB (`PutItem`), publicar en SNS (`Publish`) y generar logs en CloudWatch, sin necesidad de guardar claves ni credenciales en el código.
- **CORS:** He configurado las cabeceras de CORS tanto en la respuesta de la Lambda como en API Gateway para permitir las llamadas desde el navegador sin bloqueos.

## Prueba por terminal (cURL)
Para probar la API de forma aislada sin usar la web:

```bash
curl -X POST [https://7jjt9avvlc.execute-api.us-east-1.amazonaws.com/dev/reservations](https://7jjt9avvlc.execute-api.us-east-1.amazonaws.com/dev/reservations) \
  -H "Content-Type: application/json" \
  -d '{"name": "Sara Marco", "email": "sara@ejemplo.com", "interest": "Ruta Luna Llena"}'