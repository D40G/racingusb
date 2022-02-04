# racingusb
When item is moved into players inventory, it will install the racing app. When it is removed, it will uninstall the app.

Dependencies: 

qb-phone
qb-inventory, aj-inventory, lj-inventory. (will work for either)

EDITS

Move racing app in qb-phone/config.lua from Config.PhoneApplications to Config.StoreApps

    ["racing"] = {
        app = "racing",
        color = "#353b48",
        icon = "fas fa-flag-checkered",
        tooltipText = "Racing",
        job = false,
        blockedjobs = {},
        slot = 14,
        Alerts = 0,
    },

In qb-phone/client/main.lua,

    RegisterNetEvent('phone:client:InstallApplication', function(app, cb)
        local ApplicationData = Config.StoreApps[app]
        TriggerServerEvent('qb-phone:server:InstallApplication', ApplicationData)
    end)
    
    ----------
    
    RegisterNetEvent("phone:client:hasUsb")
    AddEventHandler("phone:client:hasUsb", function()
    	local ped = PlayerPedId()
	QBCore.Functions.TriggerCallback('qb-phone:hasRacingUsb', function(hasItems)
		if hasItems then
            		TriggerEvent('phone:client:InstallApplication', 'racing')
        	else
            		QBCore.Functions.TriggerCallback('qb-phone:server:GetPhoneData', function(pData)
               			PlayerJob = QBCore.Functions.GetPlayerData().job
                		PhoneData.PlayerData = QBCore.Functions.GetPlayerData()
                		local PhoneMeta = PhoneData.PlayerData.metadata["phone"]
                		PhoneData.MetaData = PhoneMeta
        
                		if pData.InstalledApps ~= nil and next(pData.InstalledApps) ~= nil then
                   			for k, v in pairs(pData.InstalledApps) do
                        			TriggerServerEvent('qb-phone:server:RemoveInstallation', v.app)
                        			Config.PhoneApplications['racing'] = nil
                    			end
                		end
           		end)
	  	end
	  end)
    end)
    
Add this snippet to qb-phone/server/main.lua

    QBCore.Functions.CreateCallback('qb-phone:hasRacingUsb', function(source, cb)
      	local src = source
     	local Player = QBCore.Functions.GetPlayer(src)
	local hasItems = Player.Functions.GetItemByName("racingusb")
 
	    	if hasItems then
          		cb(true)
     		else
          		cb(false)
      		end
    end)
    

I use aj-inventory and have edited it quite a bit.

replace the event for DropItemAnim with this

    RegisterNetEvent('inventory:client:DropItemAnim', function()
        local ped = PlayerPedId()
        SendNUIMessage({
            action = "close",
        })
        RequestAnimDict("pickup_object")
        while not HasAnimDictLoaded("pickup_object") do
            Wait(7)
        end
        TaskPlayAnim(ped, "pickup_object" ,"pickup_low" ,8.0, -8.0, -1, 1, 0, false, false, false )
        Wait(2000)
        ClearPedTasks(ped)
        TriggerEvent('phone:client:hasUsb')
    end)

replace the current RegisterCommand for inventory with this

    RegisterCommand('inventory', function()
      if not isCrafting and not inInventory then
        QBCore.Functions.GetPlayerData(function(PlayerData)
            if not PlayerData.metadata["isdead"] and not PlayerData.metadata["inlaststand"] and not PlayerData.metadata["ishandcuffed"] and not IsPauseMenuActive() then
                local ped = PlayerPedId()
                local curVeh = nil
                local VendingMachine = GetClosestVending()

                if IsPedInAnyVehicle(ped) then -- Is Player In Vehicle
                    local vehicle = GetVehiclePedIsIn(ped, false)
                    CurrentGlovebox = QBCore.Functions.GetPlate(vehicle)
                    curVeh = vehicle
                    CurrentVehicle = nil
                else
                    local vehicle = QBCore.Functions.GetClosestVehicle()
                    if vehicle ~= 0 and vehicle ~= nil then
                        local pos = GetEntityCoords(ped)
                        local trunkpos = GetOffsetFromEntityInWorldCoords(vehicle, 0, -2.5, 0)
                        if (IsBackEngine(GetEntityModel(vehicle))) then
                            trunkpos = GetOffsetFromEntityInWorldCoords(vehicle, 0, 2.5, 0)
                        end
                        if #(pos - trunkpos) < 2.0 and not IsPedInAnyVehicle(ped) then
                            if GetVehicleDoorLockStatus(vehicle) < 2 then
                                CurrentVehicle = QBCore.Functions.GetPlate(vehicle)
                                curVeh = vehicle
                                CurrentGlovebox = nil
                            else
                                QBCore.Functions.Notify("Vehicle Locked", "error")
                                return
                            end
                        else
                            CurrentVehicle = nil
                        end
                    else
                        CurrentVehicle = nil
                    end
                end

                if CurrentVehicle ~= nil then		-- Trunk
                    local vehicleClass = GetVehicleClass(curVeh)
                    local maxweight = 0
                    local slots = 0
                    if vehicleClass == 0 then
                        maxweight = 38000
                        slots = 30
                    elseif vehicleClass == 1 then
                        maxweight = 50000
                        slots = 40
                    elseif vehicleClass == 2 then
                        maxweight = 75000
                        slots = 50
                    elseif vehicleClass == 3 then
                        maxweight = 42000
                        slots = 35
                    elseif vehicleClass == 4 then
                        maxweight = 38000
                        slots = 30
                    elseif vehicleClass == 5 then
                        maxweight = 30000
                        slots = 25
                    elseif vehicleClass == 6 then
                        maxweight = 30000
                        slots = 25
                    elseif vehicleClass == 7 then
                        maxweight = 30000
                        slots = 25
                    elseif vehicleClass == 8 then
                        maxweight = 15000
                        slots = 15
                    elseif vehicleClass == 9 then
                        maxweight = 60000
                        slots = 35
                    elseif vehicleClass == 12 then
                        maxweight = 120000
                        slots = 35
		            elseif vehicleClass == 13 then
                        maxweight = 0
                        slots = 0
                    elseif vehicleClass == 14 then
                        maxweight = 120000
                        slots = 50
                    elseif vehicleClass == 15 then
                        maxweight = 120000
                        slots = 50
                    elseif vehicleClass == 16 then
                        maxweight = 120000
                        slots = 50
                    else
                        maxweight = 60000
                        slots = 35
                    end
                    local other = {
                        maxweight = maxweight,
                        slots = slots,
                    }
                    TriggerServerEvent("inventory:server:OpenInventory", "trunk", CurrentVehicle, other)
                    TriggerEvent('phone:client:hasUsb')
                    OpenTrunk()
                elseif CurrentGlovebox ~= nil then
                    TriggerServerEvent("inventory:server:OpenInventory", "glovebox", CurrentGlovebox)
                    TriggerEvent('phone:client:hasUsb')
                elseif CurrentDrop ~= 0 then
                    TriggerServerEvent("inventory:server:OpenInventory", "drop", CurrentDrop)
                    TriggerEvent('phone:client:hasUsb')
                elseif VendingMachine ~= nil then
                    local ShopItems = {}
                    ShopItems.label = "Vending Machine"
                    ShopItems.items = Config.VendingItem
                    ShopItems.slots = #Config.VendingItem
                    TriggerServerEvent("inventory:server:OpenInventory", "shop", "Vendingshop_"..math.random(1, 99), ShopItems)
                else
                    openAnim()
                    TriggerServerEvent("inventory:server:OpenInventory")
                    TriggerEvent('phone:client:hasUsb')
                end
            end
        end)
      end
    end)
    
Then replace RegisterNUICallback("CloseInventory") with this

    RegisterNUICallback("CloseInventory", function(data, cb)
      if currentOtherInventory == "none-inv" then
          CurrentDrop = 0
          CurrentVehicle = nil
          CurrentGlovebox = nil
          CurrentStash = nil
          SetNuiFocus(false, false)
          inInventory = false
          ClearPedTasks(PlayerPedId())
          return
      end
      if CurrentVehicle ~= nil then
          CloseTrunk()
          TriggerServerEvent("inventory:server:SaveInventory", "trunk", CurrentVehicle)
          TriggerEvent('phone:client:hasUsb')
          CurrentVehicle = nil
      elseif CurrentGlovebox ~= nil then
          TriggerServerEvent("inventory:server:SaveInventory", "glovebox", CurrentGlovebox)
          TriggerEvent('phone:client:hasUsb')
          CurrentGlovebox = nil
      elseif CurrentStash ~= nil then
          TriggerServerEvent("inventory:server:SaveInventory", "stash", CurrentStash)
          TriggerEvent('phone:client:hasUsb')
          CurrentStash = nil
      else
          TriggerServerEvent("inventory:server:SaveInventory", "drop", CurrentDrop)
          TriggerEvent('phone:client:hasUsb')
          CurrentDrop = 0
      end
      SetNuiFocus(false, false)
      inInventory = false
    end)
    
    
 This is the shared/items.lua that i use for the racingusb

 		["racingusb"] 					 = {["name"] = "racingusb", 					["label"] = "Racing USB", 			['weight'] = 1000, 		["type"] = "item", 		["image"] = "usb_device.png", 			["unique"] = true, 		["useable"] = false, 	['shouldClose'] = false, ["combinable"] = nil,  ["description"] = "A USB marked for police seizure"},
 
    
 
