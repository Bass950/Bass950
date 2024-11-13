
ARCHIVO_ORIGEN=$1
DESTINO_USUARIO=$2  # usuario@servidor_destino
DIRECTORIO_DESTINO=$3


read -sp "Ingrese la clave de cifrado: " CLAVE


FECHA=$(date +"%Y%m%d%H%M")
ARCHIVO_BASE64="/tmp/archivo_base64_$FECHA.txt"
ARCHIVO_CIFRADO="/tmp/archivo_cifrado_$FECHA.enc"
ARCHIVO_COMPRESO="/tmp/archivo_compreso_$FECHA.zip"
HASH_FILE="/tmp/hash_$FECHA.sha256"
LOG_FILE="/var/log/envio_informacion.log"

base64 "$ARCHIVO_ORIGEN" > "$ARCHIVO_BASE64"

openssl enc -aes-256-cbc -salt -in "$ARCHIVO_BASE64" -out "$ARCHIVO_CIFRADO" -k "$CLAVE"

zip -j "$ARCHIVO_COMPRESO" "$ARCHIVO_CIFRADO"

openssl dgst -sha256 "$ARCHIVO_COMPRESO" > "$HASH_FILE"

scp "$ARCHIVO_COMPRESO" "$HASH_FILE" "$DESTINO_USUARIO:$DIRECTORIO_DESTINO"
if [[ $? -eq 0 ]]; then
    echo "[$FECHA] Transferencia exitosa a $DESTINO_USUARIO" >> "$LOG_FILE"
else
    echo "[$FECHA] Error en la transferencia a $DESTINO_USUARIO" >> "$LOG_FILE"
fi

rm "$ARCHIVO_BASE64" "$ARCHIVO_CIFRADO" "$ARCHIVO_COMPRESO" "$HASH_FILE"

echo "Proceso de envío completado. Revisa el log en $LOG_FILE para detalles."
