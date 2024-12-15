# PROYECTO Análisis de redes
```bash
#!/bin/bash

PURPLE='\033[0;35m'    NC='\033[0m'  

ping -c 1 google.com &> /dev/null

if [ $? -eq 0 ]; then
  echo -e "${PURPLE}Conectividad a Internet: OK${NC}"
else
  echo -e "${PURPLE}Sin conectividad a Internet${NC}"
  exit 1
fi


read -p "Ingresa la red en formato CIDR (por ejemplo, 192.168.1.0/24): " NETWORK


USER=$(whoami) OUTPUT_FILE="analisis_red_${USER}.txt"


{ 
  echo -e "${PURPLE}Informe del análisis de red${NC}"
  echo -e "${PURPLE}Red objetivo: $NETWORK${NC}"
  echo "-----------------------------------" 
} > "$OUTPUT_FILE"


echo -e "${PURPLE}Realizando escaneo $NETWORK...${NC}"

mapfile -t HOSTS < <(nmap -sn "$NETWORK" | awk '/report for/ {print $NF}')


{ 
  echo -e "${PURPLE}Equipos:${NC}" 
} >> "$OUTPUT_FILE"


for HOST in "${HOSTS[@]}"; do

  { 
    echo -e "${PURPLE}Dispositivo: $HOST${NC}" 
  } >> "$OUTPUT_FILE"


  OPEN_PORTS=$(nc -zv -w 1 "$HOST" 1-1024 2>&1 | awk '/succeeded/ {print $3}' | tr -d ':')

  if [[ -n "$OPEN_PORTS" ]]; then 
    echo -e "  ${PURPLE}Puertos abiertos: $OPEN_PORTS${NC}" >> "$OUTPUT_FILE"
  else 
    echo -e "  ${PURPLE}No hay puertos abiertos detectados${NC}" >> "$OUTPUT_FILE"
  fi

  TTL=$(ping -c 1 "$HOST" | awk -F 'ttl=' '/ttl/ {print $2}' | awk '{print $1}')


  case "$TTL" in
    64) OS="Linux" ;;
    128) OS="Windows" ;;
    *) OS="No identificado" ;;
  esac

  echo -e "  ${PURPLE}Sistema operativo : $OS${NC}" >> "$OUTPUT_FILE"

done


echo -e "${PURPLE}Análisis completado. resultados guardados en $OUTPUT_FILE${NC}"
```
