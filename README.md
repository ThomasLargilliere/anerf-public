# anerf-rpg
# Fonctionnement
Voici le fonctionnement d'un serveur FiveM :
Il y a un côté "client" et un côté "serveur", le côté "client" c'est vous, les joueurs, le côté "serveur" c'est l'ordinateur sur lequel est hébergé le serveur.

Il faut aussi prendre en compte une chose (surement la plus importante), la base de données. Vos données sont stockés (skin, position, inventaire, ...) tous est stockés.

Les serveurs gèrent ça de différente manière, la plus commune est la suivante :
Des saves tout les X temps (par exemple toutes les 15 minutes) et une save du joueur quand il se déconnecte.

Tout ça se fait sur le serveur, donc pendant que les gens jouent le serveur est utilisé non pas pour envoyer des informations aux joueurs mais pour en sauvegarder.

J'ai mis en place un système pour essayer de gérer ça de manière beaucoup plus fluide.
J'utilise ce qu'on appelle des APIs.

Le serveur "FiveM" envoie des informations à l'ordinateur et c'est l'ordinateur qui les traites (en utilisant du python) et non pas le serveur FiveM.

L'avantage de ça c'est que si par exemple sur l'ordinateur le serveur FiveM utilise 16G de Ram et 10% du processeur, il est censé les utilisés aussi pour traiter les données, là non, c'est l'ordinateur donc ce n'est pas la ram ou le processeur attribué au serveur qui fait les calculs et les enregistrements.

De plus, au lieu d'utiliser une base de données MySQL on utilise à la fois MariaDB (qui est comme MySQL en beaucoup plus rapide) ainsi que MongoDB qui est une base de données NoSQL par conséquent beaucoup plus rapide aussi ainsiqu que pratique notamment pour le stockage des formats comme le JSON.

# Technique
## API
### CLIENT
**REQUEST**

*GET*
```
Keys.Register("F3", "F3", "Test", function()
	exports.spyoo_base:GetApi(
		'GET',
		{
			url = 'user/position/steam:11000010ceeecfa',
			Callback = 'spyoo_inventory:testCb'
		}
	)
end)
```

*POST*
```
Keys.Register("F4", "F4", "Test", function()
	exports.spyoo_base:GetApi(
		'POST',
		{
			url = 'user/position/steam:11000010ceeecfa',
			Callback = 'spyoo_inventory:testCb2',
			position = PlayerCoords
		}
	)
end)
```
**RESPONSE**

*GET*
```
RegisterNetEvent('spyoo_inventory:testCb')
AddEventHandler('spyoo_inventory:testCb', function(response)
	print(json.encode(response))
end)
```

*POST*
```
RegisterNetEvent('spyoo_inventory:testCb2')
AddEventHandler('spyoo_inventory:testCb2', function(response)
	print(json.encode(response))
end)
```
### SERVEUR
**REQUEST / RESPONSE**

*GET*
```
result = GetApi(
	{
		type = 'GET',
		url = 'user/position/' .. steamIdentifier
	}
)
```

*POST*
```
result = GetApi(
	{
		type = 'POST',
		url = 'user/connexion/' .. steamIdentifier,
		config = Config
	}
)
```


## ASYNC
**CLIENT**
```
local response = GetData('getInfoSpawn', {})
```

**SERVEUR**
```
AddEventHandler("getInfoSpawn", function(data)
    local _source = data.source
    local steamId = GetSteamId(_source)
    info = {}
    status = 'success'

    info.user = GetApi(
        {
            type = 'GET',
            url = 'user/position/' .. steamId,
            config = Config
        }
    )

    info.status = status
    info.event = data.event
    info.source = data.source
    TriggerEvent('getInfo', info)
    TriggerEvent('spyoo_skin:spawn', data.source)
    TriggerEvent('spyoo_inventory:spawn', data.source)
end)
```
