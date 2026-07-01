# 🖥️ Virtuális gép létrehozása és kezelése Azure CLI-vel

Ez a leírás bemutatja, hogyan hozhatsz létre, kérdezhetsz le és kezelhetsz virtuális gépeket, valamint a hozzájuk tartozó hálózati erőforrásokat az Azure CLI segítségével.

---

## 🔐 1. Bejelentkezés és Alapvető Lekérdezések

Első lépésként jelentkezz be az Azure fiókodba a bérlői azonosítód (Tenant ID) megadásával:

```bash
az login --tenant "TENANT_AZONOSITO"
az login --tenant XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX
```

### 📋 Környezet ellenőrzése
Miután bejelentkeztél, az alábbi parancsokkal térképezheted fel a meglévő környezetedet:

* **Aktuális előfizetések megtekintése:**
  ```bash
  az account list --output table
  ```
* **Erőforráscsoportok (Resource Groups) listázása:**
  ```bash
  az group list --output table
  ```
<img width="1256" height="470" alt="image" src="https://github.com/user-attachments/assets/2b38b09a-3598-44c6-b63c-2953bb8df5b5" />

---

## 🌐 2. Hálózati Infrastruktúra Kezelése (Opcionális)

Amennyiben saját hálózatot szeretnél kialakítani a virtuális gép alá, használd az alábbi parancsokat.

### Virtuális hálózatok (VNet)
* **Összes VNet lekérdezése:**
  ```bash
  az network vnet list --output table
  ```
* **Szűrés egy adott erőforráscsoportra:**
  ```bash
  az network vnet list --resource-group "EROFORRASCSOPORT_NEVE" --output table
  ```
* **Új VNet létrehozása:**
  ```bash
  az network vnet create --resource-group "EROFORRASCSOPORT_NEVE" --name "VNet_neve"
  ```

### Alhálózatok (Subnet)
* **Létező alhálózatok listázása:**
  ```bash
  az network vnet subnet list --resource-group "EROFORRASCSOPORT_NEVE" --vnet-name "VNet_neve" --output table
  ```
* **Új alhálózat létrehozása (Kötelező megadni a címteret!):**
  ```bash
  az network vnet subnet create --resource-group "EROFORRASCSOPORT_NEVE" --vnet-name "VNet_neve" --name "SubNet_neve" --address-prefixes 10.0.0.0/24
  ```

---

## 🚀 3. Virtuális Gép Létrehozása

Ha nem szeretnél a VNet és Subnet manuális beállításaival bajlódni, az Azure automatikusan létrehozza azokat a gép indításakor.

### Előkészületek

* **Erőforráscsoport létrehozása**
   ```bash
  az group create --name "EROFORRASCSOPORT_NEVE" --location "pl: swedencentral"
  ```

* **Elérhető régiók (Location) listája:**
  ```bash
  az account list-locations --output table
  ```
* **Elérhető gépméretek lekérdezése az adott régióban:** 
  ```bash
  az vm list-sizes --location "pl: swedencentral" --output table
  ```
  
* **Elérhető operációs rendszerek adott régióban:**
  ```bash
  az vm image list --location "pl: swedencentral" --output table
  ```


### Új Virtuális Gép létrehozása
Az alábbi parancs létrehoz egy alapértelmezett virtuális gépet:

```bash
az vm create \
  --resource-group "EROFORRASCSOPORT_NEVE" \
  --name "VM_neve" \
  --size "Standard_B2s" \
  --public-ip-sku Standard \
  --admin-username "azureuser" \
  --admin-password "BiztonsagosJelszo1234@!"
```

```bash
az vm create --resource-group "EROFORRASCSOPORT_NEVE" --name "VM_neve" --size "Standard_B2s" --public-ip-sku Standard --admin-username "azureuser" --admin-password "BiztonsagosJelszo1234@!"
```

<img width="1720" height="311" alt="image" src="https://github.com/user-attachments/assets/6e3ec876-8490-4ac5-909d-69c37a2d18c4" />



Ha a meglévő vNrt-hez szeretnéd illeszteni azt így teheted meg:
```bash
az vm create \
  --resource-group "EROFORRASCSOPORT_NEVE" \
  --name "VM_neve" \
  --size "Standard_B2s" \
  --vnet-name "VNet_neve" \
  --subnet "Subnet_neve" \
  --public-ip-sku Standard \
  --admin-username "azureuser" \
  --admin-password "BiztonsagosJelszo1234@!"
```


## 📊 4. Operációs Rendszerek (OS Images) Választása

Alapértelmezett gép helyett saját Linux vagy Windows rendszert is választhatsz a `--image` kapcsoló használatával.

* **Elérhető népszerű lemezképek listázása:**
  ```bash
  az vm image list --output table
  ```
> 💡 **Tipp:** A választás során a táblázatban megjelenő **UrnAlias** értékére kell hivatkozni (pl. `Ubuntu2204` vagy `Win2022Datacenter`).


---

## ⚙️ 5. Virtuális Gépek Kezelése és Állapota

### Lekérdezések
* **Egy erőforráscsoport összes gépének listázása:**
  ```bash
  az vm list --subscription "ELOFIZETES_NEVE" --resource-group "EROFORRASCSOPORT_NEVE" --output table
  ```
* **Egy konkrét gép részletes adatainak megtekintése:**
  ```bash
  az vm show --subscription "ELOFIZETES_NEVE" --resource-group "EROFORRASCSOPORT_NEVE" --name "VM_neve"
  ```
* **A gépek futási állapotának (State) ellenőrzése:**
  ```bash
  az vm list --resource-group "EROFORRASCSOPORT_NEVE" --show-details --output table
  ```
* **A gép nyilvános IP címének lekérdezése:**
  ```bash
  az vm list-ip-addresses --resource-group "EROFORRASCSOPORT_NEVE" --name "VM_neve" --output table
  ```

<img width="1680" height="607" alt="image" src="https://github.com/user-attachments/assets/c9727d41-e1b5-4318-8d32-65ae82fbe402" />

  

### Életciklus parancsok
* **Virtuális gép elindítása:**
  ```bash
  az vm start --resource-group "EROFORRASCSOPORT_NEVE" --name "VM_neve"
  ```
* **Virtuális gép leállítása (Felszabadítás/Deallocate - így nem generál költséget):**
  ```bash
  az vm deallocate --resource-group "EROFORRASCSOPORT_NEVE" --name "VM_neve"
  ```

<img width="860" height="192" alt="image" src="https://github.com/user-attachments/assets/edb6a9a9-77ec-4fd2-b73a-b8872684fc8c" />



---

## 🗑️ 6. Erőforrások Törlése

* **Csak a virtuális gép törlése:**
  ```bash
  az vm delete --resource-group "EROFORRASCSOPORT_NEVE" --name "VM_neve" --yes
  ```
> ⚠️ **Figyelem:** A gép törlése után a hozzá tartozó hálózati kártya (NIC), lemezek (Disk) és VNet a felhőben maradnak és költséget okozhatnak.

* **A legtisztább törlés (A teljes erőforráscsoport megsemmisítése mindennel együtt):**
  ```bash
  az group delete --name "EROFORRASCSOPORT_NEVE" --yes
  ```
