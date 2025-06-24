
alias opt_kill_points     = script_option[0]
alias opt_jug_kill_points = script_option[1] -- points for killing while you are the juggernaut
alias opt_safe_time       = script_option[2]

alias juggernaut_traits     = script_traits[0]
alias new_juggernaut_traits = script_traits[1]

-- forge labels
alias all_gravity_hammers = 2
alias all_energy_swords   = 3 -- did Bungie plan to have a weapon option, and scrap it?
alias all_skulls          = 4 -- dunno about this one. maybe attaching skulls to the juggernaut as decoration?
alias all_health_packs    = 5

alias temp_int_00          = global.number[0] -- unused
alias temp_int_01          = global.number[1] -- unused
alias temp_int_02          = global.number[2]
alias juggernaut_flames    = global.object[0]
alias temp_obj_00          = global.object[1]
alias temp_obj_01          = global.object[2]
alias current_juggernaut   = global.player[0]
alias temp_plr_00          = global.player[1]
alias temp_plr_01          = global.player[2]
alias temp_plr_02          = global.player[3]
alias safe_timer           = global.timer[0]
alias initial_juggernaut_delay = global.timer[1] -- also applies for players promoted to Juggernaut after the previous Juggernaut's suicide
alias marked_for_delete    = object.number[0] -- only for gravity hammers; we want to delete the jug's weapon when they die
alias announced_game_start = player.number[0]
alias juggernaut_time_inc  = player.timer[0] -- a 1-second timer used to know when to increment the "Juggernaut Time" stat

alias team_normal     = team[0]
alias team_juggernaut = team[1]

declare temp_int_00        with network priority local
declare temp_int_01        with network priority local
declare temp_int_02        with network priority local
declare juggernaut_flames  with network priority low
declare temp_obj_00        with network priority local
declare temp_obj_01        with network priority local
declare current_juggernaut with network priority low
declare temp_plr_00        with network priority local
declare temp_plr_01        with network priority local
declare temp_plr_02        with network priority local
declare safe_timer = opt_safe_time
declare global.timer[1] = 5
declare player.announced_game_start with network priority low
declare player.juggernaut_time_inc = 1
declare object.marked_for_delete with network priority local

alias juggernaut_time = player.script_stat[0]

for each player do -- loadout palettes
   if current_player.is_elite() then 
      current_player.set_loadout_palette(elite_tier_1)
   end
   if not current_player.is_elite() then 
      current_player.set_loadout_palette(spartan_tier_1)
   end
end

do -- timer rates
   safe_timer.set_rate(-100%)
   initial_juggernaut_delay.set_rate(-100%)
end

for each player do -- announce game start and handle round card
   if current_player.announced_game_start == 0 then 
      if game.score_to_win != 0 then 
         current_player.set_round_card_title("Kill the Juggernaut.\r\n%n points to win.", game.score_to_win)
      end
      if game.score_to_win == 0 then 
         current_player.set_round_card_title("Kill the Juggernaut.")
      end
      send_incident(juggernaut_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

for each player do -- manage "Juggenaut Time" stat
   current_player.juggernaut_time_inc.set_rate(0%)
   if current_juggernaut == current_player then 
      temp_obj_00 = current_juggernaut.biped
      if not temp_obj_00 == no_object then 
         current_player.juggernaut_time_inc.set_rate(-100%)
         if current_player.juggernaut_time_inc.is_zero() then 
            current_player.juggernaut_time += 1
            current_player.juggernaut_time_inc.reset()
         end
      end
   end
end

if initial_juggernaut_delay.is_zero() and not current_juggernaut == no_player and not current_juggernaut.is_not_respawning() then
   --
   -- If the juggernaut is waiting to respawn (because they have died somehow) and a 
   -- minimum duration has elapsed, then revoke juggernaut status and reset the timer. 
   -- We will end up assigning a new juggernaut when the timer next hits zero.
   --
   -- This has a few consequences:
   --
   --  - When a Juggernaut dies by suicide, they lose their status immediately, but 
   --    there is a delay before a new Juggernaut is selected.
   --
   --  - There is a similar delay before the selection of the first Juggernaut.
   --
   current_juggernaut = no_player
   initial_juggernaut_delay.reset()
   initial_juggernaut_delay.set_rate(-100%)
end

on host migration: do
   initial_juggernaut_delay.reset()
   initial_juggernaut_delay.set_rate(-100%)
end

for each player do -- handle deaths
   alias jugger = temp_plr_00
   alias killer = temp_plr_01
   alias victim = temp_plr_02
   alias is_suicide = temp_int_02
   --
   jugger = current_juggernaut
   killer = no_player
   victim = current_player
   is_suicide = 0
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      killer = current_player.try_get_killer()
      if victim == jugger and killer == victim or killer == no_player then -- suicide/guardians/etc.
         is_suicide = 1
      end
      if not is_suicide == 1 and killer == jugger then  -- juggernaut killed someone
         killer.score += opt_jug_kill_points
         send_incident(kill_as_juggernaut, killer, victim)
      end
      if current_player == jugger and not is_suicide == 1 then -- someone killed juggernaut
         killer.score += opt_kill_points
         send_incident(juggernaut_kill, killer, jugger)
         jugger.set_round_card_title("Kill the Juggernaut.\r\n%n points to win.", game.score_to_win)
         current_juggernaut = killer
         send_incident(new_juggernaut, current_juggernaut, all_players)
         safe_timer.reset()
         safe_timer.set_rate(-100%)
         current_juggernaut.biped.remove_weapon(secondary, false)
         current_juggernaut.biped.remove_weapon(primary, false)
         current_juggernaut.biped.add_weapon(gravity_hammer, force)
         current_juggernaut.biped.shields   = 200
         current_juggernaut.frag_grenades   = 0
         current_juggernaut.plasma_grenades = 0
         juggernaut_flames.delete()
         juggernaut_flames = current_juggernaut.biped.place_at_me(particle_emitter_fire, none, never_garbage_collect, 0, 0, 0, none)
         juggernaut_flames.attach_to(current_juggernaut.biped, 0, 0, 5, absolute)
      end
   end
end

for each player randomly do -- assign initial juggernaut (and post-suicide juggernaut) to a random player
   if initial_juggernaut_delay.is_zero() then 
      temp_plr_00 = current_juggernaut
      temp_obj_00 = current_juggernaut.biped
      if current_juggernaut == no_player and not current_player == temp_plr_00 then 
         temp_obj_01 = current_player.biped
         if not temp_obj_01 == no_object and not current_player.biped.is_out_of_bounds() then 
            current_juggernaut = current_player
            send_incident(new_juggernaut, current_juggernaut, all_players)
            current_juggernaut.biped.remove_weapon(secondary, false)
            current_juggernaut.biped.remove_weapon(primary, false)
            current_juggernaut.biped.add_weapon(gravity_hammer, force)
            current_juggernaut.biped.shields   = 200
            current_juggernaut.frag_grenades   = 0
            current_juggernaut.plasma_grenades = 0
            safe_timer.reset()
            safe_timer.set_rate(-100%)
            juggernaut_flames.delete()
            juggernaut_flames = current_juggernaut.biped.place_at_me(particle_emitter_fire, none, never_garbage_collect, 0, 0, 0, none)
            juggernaut_flames.attach_to(current_juggernaut.biped, 0, 0, 5, absolute)
         end
      end
   end
end

do -- handle juggernaut traits, waypoint, and round card
   for each player do
      current_player.biped.set_waypoint_icon(none)
      current_player.biped.set_waypoint_priority(low)
   end
   current_juggernaut.apply_traits(juggernaut_traits)
   current_juggernaut.biped.set_waypoint_icon(bullseye)
   current_juggernaut.biped.set_waypoint_priority(high)
   current_juggernaut.set_round_card_title("You are the Juggernaut")
   if not safe_timer.is_zero() then 
      current_juggernaut.apply_traits(new_juggernaut_traits)
      current_juggernaut.biped.set_waypoint_icon(skull)
   end
end

for each object with label all_gravity_hammers do -- queue gravity hammers for deletion unless the current juggernaut is holding them
   alias current_carrier = temp_plr_00
   --
   current_object.marked_for_delete = 1
   current_carrier = no_player
   current_carrier = current_object.try_get_carrier()
   if current_carrier == current_juggernaut then 
      current_object.marked_for_delete = 0
   end
end

-- Manage Juggernaut's currently-equipped weapon:
if not current_juggernaut == no_player then
   alias current_weapon = temp_obj_00
   --
   current_weapon = no_object
   current_weapon = current_juggernaut.try_get_weapon(secondary)
   if not current_weapon == no_object and not current_weapon.is_of_type(gravity_hammer) then 
      current_juggernaut.biped.remove_weapon(primary, false)
   end
end
if not current_juggernaut == no_player then
   alias current_weapon = temp_obj_00
   --
   current_weapon = no_object
   current_weapon = current_juggernaut.try_get_weapon(primary)
   if not current_weapon.is_of_type(gravity_hammer) then
      current_juggernaut.biped.add_weapon(gravity_hammer, force) -- "force" is needed to override Weapon Pickup: Disabled
   end
end

for each object with label all_gravity_hammers do -- delete queued gravity hammers
   if current_object.marked_for_delete == 1 then 
      current_object.delete()
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each player do -- manage player teams
   current_player.team = team_normal
   if current_player == current_juggernaut then 
      current_player.team = team_juggernaut
   end
end

for each player do -- if the juggernaut ever lacks a biped, revoke their status
   if not current_juggernaut == no_player then 
      temp_obj_00 = current_juggernaut.biped
      if temp_obj_00 == no_object then 
         current_juggernaut = no_player
      end
   end
end

for each object with label all_health_packs do -- delete all health packs
   current_object.delete()
end

for each player do -- players who are waiting to respawn should not have an alliance status
   temp_obj_00 = current_player.biped
   if temp_obj_00 == no_object then 
      current_player.team = no_team
   end
end
