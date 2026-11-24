# Resumen técnico — Conectividad AWS ELK ↔ HML Contabo

**Proyecto:** Brosco — Auditoría SSH  
**Fecha:** 24/11/2026  
**Estado:** Implementado y validado

## 1. Objetivo

Habilitar la conectividad SSH desde el servidor AWS ELK hacia el servidor HML de Contabo, utilizando la infraestructura VPN y el gateway `new-phoebe` existente.

## 2. Arquitectura

```
AWS ELK (172.31.67.8)
        |
        | VPN IPsec existente
        v
new-phoebe (192.168.68.2:2260)
        |
        | DNAT + SNAT
        v
Contabo HML (10.69.0.20:6722)
        |
        v
      SSH
```

## 3. Configuración implementada

En `new-phoebe` se incorporaron dos reglas NAT:

**DNAT:** redirige las conexiones provenientes de AWS ELK hacia el servicio SSH de HML.

```
192.168.68.2:2260 → 10.69.0.20:6722
```

**SNAT:** establece `192.168.68.2` como dirección de origen para permitir el retorno correcto del tráfico.

Las reglas se incorporaron al archivo persistente:

```
/etc/nftables.d/47-nat-brosco-lan.nft
```

Se aplicaron mediante `nft` sin recargar completamente el firewall, evitando afectar las conexiones existentes.

No fue necesario modificar la VPN IPsec.

## 4. Validación

Desde AWS ELK:

```
nc -vz -w 5 192.168.68.2 2260
```

Resultado:

```
Connection to 192.168.68.2 2260 port [tcp/*] succeeded!
```

También se verificó que los contadores de las reglas DNAT y SNAT registraban tráfico.

Posteriormente se ejecutó exitosamente una sincronización SSH/rsync hacia HML, confirmando la conectividad de extremo a extremo.

## 5. Resultado final

**Conectividad AWS ELK → HML operativa y persistente.**

La solución reutiliza la VPN existente, mantiene separada la conexión de DEV (`2240`) y HML (`2260`) y permite incorporar los logs SSH de HML al recolector centralizado sin modificar la arquitectura de red existente.