
alias opt_zombie_count        = script_option[0]
alias opt_last_man_standing   = script_option[1]
alias opt_safe_havens         = script_option[2]
alias opt_haven_timer         = script_option[3]
alias opt_zombie_win_points   = script_option[4]
alias opt_survivor_win_points = script_option[5]
alias opt_haven_points        = script_option[6]
alias opt_kill_points         = script_option[7]
alias opt_suicide_points      = script_option[8]
alias opt_betrayal_points     = script_option[9]
alias opt_zombie_kill_points  = script_option[10]
alias opt_last_man_points     = script_option[11]
alias opt_suicides_to_zombies = script_option[12]
alias opt_alpha_zombies       = script_option[13]

alias infected_traits     = script_traits[0]
alias alpha_zombie_traits = script_traits[1]
alias last_man_traits     = script_traits[2]
alias haven_traits        = script_traits[3]

alias ui_haven_timer = script_widget[0]

alias survival_time = player.script_stat[0]
alias infections    = player.script_stat[1]

-- Unnamed Forge labels:
alias all_jetpacks = 0

alias team_survivors = team[0]
alias team_zombies   = team[1]

alias announced_game_start = player.number[4]
alias announce_start_timer = player.timer[0]

alias temp_int_00    = global.number[0]
alias chose_last_man = global.number[1] -- presumably used to prevent joins-in-progress from complicating things
alias is_not_first_haven = global.number[2]
alias temp_int_01    = global.number[4]
alias temp_obj_00    = global.object[0]
alias active_haven   = global.object[1]
alias previous_haven = global.object[2]
alias temp_plr_00       = global.player[0]
alias temp_plr_01       = global.player[1]
alias haven_timer       = global.timer[0]
alias match_start_timer = global.timer[1] -- zombies cannot win until this timer runs out
alias haven_timer_started = object.number[0] -- only starts when a survivor enters the Haven
alias ui_haven_timer      = object.timer[0]
alias is_zombie    = player.number[0]
alias is_last_man  = player.number[1]
alias is_in_haven  = player.number[2] -- survivors only
alias is_alpha     = player.number[3]
alias ach_emergency_room_count      = player.number[5]
alias survival_time_update_interval = player.timer[1]
alias infection_timer = player.timer[2] -- attempted failsafe for betrayals, but the precise problems it's meant to address are not clear

declare temp_int_00        with network priority local
declare chose_last_man     with network priority low
declare is_not_first_haven with network priority local
declare global.number[3]   with network priority low -- unused
declare temp_int_01        with network priority local
declare temp_obj_00    with network priority local
declare active_haven   with network priority low
declare previous_haven with network priority local
declare temp_plr_00 with network priority local
declare temp_plr_01 with network priority local
declare haven_timer = opt_haven_timer
declare match_start_timer = 10
declare player.is_zombie   with network priority low
declare player.is_last_man with network priority low
declare player.is_in_haven with network priority low = 1
declare player.is_alpha    with network priority low
declare player.number[4] with network priority low -- unused
declare player.ach_emergency_room_count with network priority low
declare player.announce_start_timer = 5
declare player.survival_time_update_interval = 1
declare object.haven_timer_started with network priority low
declare object.ui_haven_timer = opt_haven_timer

for each player do -- award Dive Bomber achievement as appropriate
   alias death_mod = temp_int_00
   alias killer    = temp_plr_00
   alias killer_aa = temp_obj_00
   if current_player.killer_type_is(kill) then 
      killer    = no_player
      killer    = current_player.try_get_killer()
      death_mod = 0
      death_mod = current_player.try_get_death_damage_mod()
      if death_mod == enums.damage_reporting_modifier.assassination then 
         killer_aa = no_object
         killer_aa = killer.try_get_armor_ability()
         if killer_aa.has_forge_label(all_jetpacks) and killer_aa.is_in_use() then 
            send_incident(dlc_achieve_2, killer, killer, 65)
         end
      end
   end
end

for each player do -- award From Hell's Heart achievement as appropriate
   alias death_mod = temp_int_00
   alias killer    = temp_plr_00
   if current_player.killer_type_is(kill) then 
      death_mod = 0
      death_mod = current_player.try_get_death_damage_mod()
      if death_mod == enums.damage_reporting_modifier.sticky then 
         killer = no_player
         killer = current_player.try_get_killer()
         if temp_plr_00.killer_type_is(suicide) then 
            send_incident(dlc_achieve_2, current_player, current_player, 68)
         end
      end
   end
end

for each player do -- loadout palettes
   if current_player.is_zombie != 1 then 
      if current_player.is_elite() then 
         current_player.set_loadout_palette(elite_tier_1)
      end
      if not current_player.is_elite() then 
         current_player.set_loadout_palette(spartan_tier_1)
      end
   end
   if current_player.is_zombie == 1 then 
      current_player.infection_timer.set_rate(-100%)
      if current_player.is_elite() then 
         current_player.set_loadout_palette(elite_tier_2)
      end
      if not current_player.is_elite() then 
         current_player.set_loadout_palette(spartan_tier_2)
      end
   end
end

do -- maintain minimum zombie count
   alias alpha_count = temp_int_00
   alias max_zombies = temp_int_01 -- number of players in the match, minus one
   --
   alpha_count = 0
   max_zombies = -1
   for each player do
      max_zombies += 1
      if current_player.is_alpha == 1 then 
         alpha_count += 1
      end
   end
   for each player randomly do
      if  alpha_count < opt_zombie_count
      and alpha_count < max_zombies -- ensure there is always at least one survivor
      and current_player.is_alpha  != 1
      and current_player.is_zombie != 1
      then 
         current_player.is_zombie = 1
         current_player.infection_timer = 3
         current_player.is_alpha = 1
         alpha_count += 1
      end
   end
   for each player do
      if current_player.is_zombie == 1 and current_player.team != team_zombies then 
         send_incident(inf_new_zombie, current_player, no_player)
         current_player.team = team_zombies
         current_player.apply_traits(infected_traits)
         current_player.biped.kill(true)
      end
   end
end

for each player do -- initialize UI
   ui_haven_timer.set_text("Safe Haven - %s", haven_timer)
   ui_haven_timer.set_visibility(current_player, false)
end

for each player do -- round card and announce start timer
   current_player.announce_start_timer.set_rate(-100%)
   for each player do
      if current_player.team == team_survivors then 
         current_player.set_round_card_title("Defend yourself from the zombie horde!")
      end
   end
   for each player do
      if current_player.team == team_zombies then 
         current_player.set_round_card_title("Braaaaaains...")
      end
   end
end
for each player do -- announce game start
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      send_incident(infection_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

for each player do -- maintain player teams and traits
   current_player.team = team_survivors
   if current_player.is_zombie == 1 then 
      current_player.team = team_zombies
      current_player.apply_traits(infected_traits)
      if current_player.is_alpha == 1 and opt_alpha_zombies == 1 then 
         current_player.apply_traits(alpha_zombie_traits)
      end
   end
end

for each player do
   alias victim = temp_plr_00
   alias killer = temp_plr_01
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.is_last_man = 0
      victim = current_player
      killer = no_player
      killer = current_player.try_get_killer()
      if  current_player.killer_type_is(kill)
      and victim.is_zombie == 1
      and victim.is_zombie != killer.is_zombie
      then 
         killer.score += opt_kill_points
         send_incident(zombie_kill_kill, killer, victim)
      end
      if  current_player.killer_type_is(kill)
      and opt_safe_havens == 1
      and victim.is_zombie == 1
      and victim.is_zombie != killer.is_zombie
      and killer.is_in_haven == 1
      then 
         killer.score += opt_haven_points -- points for killing a zombie from inside of a Haven
      end
      if  current_player.killer_type_is(kill)
      and not killer == no_player
      and victim.is_zombie == 0
      then 
         victim.is_zombie = 1
         victim.infection_timer = 3
         send_incident(inf_new_infection, killer, victim)
         send_incident(infection_kill,    killer, victim)
         killer.score += opt_zombie_kill_points
         killer.infections += 1
         killer.ach_emergency_room_count += 1
         if killer.ach_emergency_room_count > 2 then 
            send_incident(dlc_achieve_2, killer, killer, 63) -- award Emergency Room achievement
         end
      end
      if current_player.killer_type_is(suicide) then -- handle suicides
         killer.score += opt_suicide_points
         if opt_suicides_to_zombies == 1 then 
            victim.is_zombie = 1
         end
      end
      if  current_player.killer_type_is(betrayal)
      and killer.infection_timer.is_zero()
      and victim.infection_timer.is_zero()
      and victim.is_zombie == killer.is_zombie
      then 
         killer.score += opt_betrayal_points
      end
   end
end

if game.round_timer.is_zero() and game.round_time_limit > 0 then -- reset Emergency Room progress when the round time runs out
   for each player do
      current_player.ach_emergency_room_count = 0
   end
end

if opt_safe_havens == 1 and active_haven == no_object then -- handle there being no Haven
   active_haven = get_random_object("inf_haven", previous_haven)
   if is_not_first_haven == 1 then 
      send_incident(hill_moved, all_players, all_players)
   end
   is_not_first_haven = 1
end
if opt_safe_havens == 1 and haven_timer.is_zero() then -- move the Haven
   haven_timer.set_rate(0%)
   haven_timer = opt_haven_timer
   active_haven.set_waypoint_visibility(no_one)
   active_haven.set_shape_visibility(no_one)
   active_haven.set_waypoint_timer(none)
   active_haven.haven_timer_started = 0
   previous_haven = active_haven
   active_haven = no_object
   active_haven = get_random_object("inf_haven", previous_haven)
end

do -- maintain Haven waypoint
   active_haven.set_waypoint_visibility(everyone)
   active_haven.set_waypoint_icon(crown)
   active_haven.set_shape_visibility(everyone)
   active_haven.set_waypoint_priority(high)
end

if opt_safe_havens == 1 then -- handle starting the Haven timer and managing its waypoint
   for each player do
      if  active_haven.shape_contains(current_player.biped)
      and current_player.is_zombie == 0
      and active_haven.haven_timer_started == 0
      then 
         haven_timer.set_rate(-100%)
         active_haven.haven_timer_started = 1
      end
   end
   if active_haven.haven_timer_started == 1 then 
      active_haven.ui_haven_timer = haven_timer
      active_haven.set_waypoint_timer(object.ui_haven_timer)
      if active_haven.ui_haven_timer < 6 then 
         active_haven.set_waypoint_priority(blink)
      end
   end
end

if opt_last_man_standing == 1 then -- handle Last Man Standing status
   alias survivor_count = temp_int_00
   --
   survivor_count = 0
   if chose_last_man == 0 then 
      for each player do
         if not current_player.is_zombie == 1 then
            survivor_count += 1
         end
      end
      if survivor_count == 1 then
         for each player do
            if not current_player.is_zombie == 1 then 
               current_player.apply_traits(last_man_traits)
               current_player.biped.set_waypoint_icon(skull)
               current_player.biped.set_waypoint_priority(high)
               current_player.is_last_man = 1
               current_player.score += opt_last_man_points
               send_incident(inf_last_man,  current_player, all_players)
               send_incident(dlc_achieve_2, current_player, current_player, 61) -- award the All Alone achievement
            end
         end
         chose_last_man = 1
      end
   end
end

for each player do -- Last Man Standing traits
   if current_player.is_last_man == 1 then 
      current_player.apply_traits(last_man_traits)
   end
end

for each player do -- handle player-in-Haven behavior
   ui_haven_timer.set_visibility(current_player, false)
   current_player.is_in_haven = 0
   if opt_safe_havens == 1 and active_haven.shape_contains(current_player.biped) and current_player.is_zombie == 0 then 
      current_player.is_in_haven = 1
      current_player.apply_traits(haven_traits)
      ui_haven_timer.set_visibility(current_player, true)
   end
end

do -- zombie victory win condition
   alias survivor_count = temp_int_00
   --
   match_start_timer.set_rate(-100%)
   if match_start_timer.is_zero() then 
      survivor_count = 0
      for each player do
         if current_player.is_zombie == 0 then 
            survivor_count += 1
         end
      end
      for each player do
         if survivor_count == 1 and current_player.is_zombie == 0 and current_player.killer_type_is(suicide) then 
            survivor_count = 0
         end
      end
      if survivor_count == 0 then 
         send_incident(infection_zombie_win, all_players, all_players)
         for each player do
            --
            -- Let's award points to all zombies for their victory. We need to make sure 
            -- we don't accidentally also award the newly-infected Last Man Standing, who 
            -- by definition did not contribute to a zombie victory.
            --
            if current_player.is_last_man != 1 and current_player.is_zombie == 1 then
               current_player.score += opt_zombie_win_points
            end
         end
         game.end_round()
      end
   end
end

if game.round_timer.is_zero() and game.round_time_limit > 0 then -- survivor victory win condition
   alias survivor_count = temp_int_00
   --
   survivor_count = 0
   for each player do
      if current_player.is_zombie == 0 then 
         survivor_count += 1
      end
   end
   if not survivor_count == 0 then 
      send_incident(infection_survivor_win, all_players, all_players)
      for each player do
         if current_player.is_zombie == 0 then 
            current_player.score += opt_survivor_win_points
         end
      end
      game.end_round()
   end
end

for each player do -- track Survival Time
   if current_player.is_zombie == 0 then 
      current_player.survival_time_update_interval.set_rate(-100%)
      if current_player.survival_time_update_interval.is_zero() then 
         current_player.survival_time += 1
         current_player.survival_time_update_interval.reset()
      end
   end
end
