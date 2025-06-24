
alias opt_drop_points    = script_option[0]
alias opt_drop_movement  = script_option[1]
alias opt_instant_win    = script_option[2]
alias opt_headshots_only = script_option[3]

-- Forge labels
alias all_jetpacks    = 0
alias all_armor_locks = 1
alias all_skulls      = 4

alias ui_skull_count = script_widget[0]

alias skull_carrier_traits     = script_traits[0]
alias max_skull_carrier_traits = script_traits[1]

alias drop_point_movement_timer = global.timer[0]
alias is_active           = object.number[0] -- for drop points
alias activation_cooldown = object.timer[1] -- a single goal cannot be reactivate until this many seconds after it stopped being active
alias ach_top_shot_count        = player.number[0]
alias ach_license_to_kill_count = player.number[1]
alias carried_skulls = player.number[2]
alias last_biped     = player.object[0]
alias ach_paper_beats_rock_vuln_timer = player.timer[0]
alias skull_take_announce_cooldown = player.timer[2] -- don't announce "Skull taken!" too frequently

alias announced_game_start = player.number[3]
alias announce_start_timer = player.timer[1]

alias max_skulls = player.script_stat[0]

declare global.number[0] with network priority local
declare global.number[1] with network priority low -- temporary
declare global.number[2] with network priority local
declare global.object[0] with network priority local
declare global.object[1] with network priority local
declare global.player[0] with network priority local
declare drop_point_movement_timer = opt_drop_movement
declare player.ach_top_shot_count        with network priority low
declare player.ach_license_to_kill_count with network priority low
declare player.carried_skulls            with network priority low
declare player.announced_game_start      with network priority low
declare player.last_biped                with network priority local
declare player.announce_start_timer = 5
declare player.skull_take_announce_cooldown = 3
declare object.is_active with network priority low
declare object.number[1] with network priority low -- unused?
declare object.timer[0] = 77 -- unused?
declare object.activation_cooldown = 3

for each player do -- award Dive Bomber achievement as appropriate
   if current_player.killer_type_is(kill) then 
      global.player[0] = no_player
      global.player[0] = current_player.try_get_killer()
      global.number[0] = 0
      global.number[0] = current_player.try_get_death_damage_mod()
      if global.number[0] == 2 then 
         global.object[0] = no_object
         global.object[0] = global.player[0].try_get_armor_ability()
         if global.object[0].has_forge_label(all_jetpacks) and global.object[0].is_in_use() then 
            send_incident(dlc_achieve_2, global.player[0], global.player[0], 65)
         end
      end
   end
end

for each player do -- award From Hell's Heart achievement as appropriate
   if current_player.killer_type_is(kill) then 
      global.number[0] = 0
      global.number[0] = current_player.try_get_death_damage_mod()
      if global.number[0] == 4 then 
         global.player[0] = no_player
         global.player[0] = current_player.try_get_killer()
         if global.player[0].killer_type_is(suicide) then 
            send_incident(dlc_achieve_2, current_player, current_player, 68)
         end
      end
   end
end

for each player do -- manage and award Top Shot achievement as appropriate
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.ach_top_shot_count = 0
      if current_player.killer_type_is(kill) then 
         global.player[0] = no_player
         global.player[0] = current_player.try_get_killer()
         global.number[0] = 0
         global.number[0] = current_player.try_get_death_damage_mod()
         if global.number[0] != 5 then 
            global.player[0].ach_top_shot_count = 0
         end
         if global.number[0] == 5 then 
            global.player[0].ach_top_shot_count += 1
            if global.player[0].ach_top_shot_count > 2 then 
               send_incident(dlc_achieve_2, global.player[0], global.player[0], 62)
            end
         end
      end
   end
end

for each player do -- manage and award License To Kill achievement as appropriate
   if current_player.killer_type_is(kill) then 
      global.player[0] = no_player
      global.player[0] = current_player.try_get_killer()
      global.number[0] = 0
      global.number[0] = current_player.try_get_death_damage_mod()
      global.object[0] = no_object
      global.object[0] = global.player[0].try_get_vehicle()
      if global.object[0] != no_object and global.number[0] == 3 then 
         global.player[0].ach_license_to_kill_count += 1
         if global.player[0].ach_license_to_kill_count > 4 then 
            send_incident(dlc_achieve_2, global.player[0], global.player[0], 66)
         end
      end
   end
end

for each player do -- manage timing for the Paper Beats Rock achievement
   global.object[0] = no_object
   global.object[0] = current_player.try_get_armor_ability()
   if global.object[0].has_forge_label(all_armor_locks) and global.object[0].is_in_use() then 
      current_player.ach_paper_beats_rock_vuln_timer = 4
      current_player.ach_paper_beats_rock_vuln_timer.set_rate(-100%)
   end
end
for each player do -- award Paper Beats Rock achievement as appropriate
   if current_player.killer_type_is(kill) and not current_player.ach_paper_beats_rock_vuln_timer.is_zero() then 
      global.number[0] = 0
      global.number[0] = current_player.try_get_death_damage_mod()
      if global.number[0] == 2 then 
         global.player[0] = no_player
         global.player[0] = current_player.try_get_killer()
         send_incident(dlc_achieve_2, global.player[0], global.player[0], 60)
      end
   end
end

if game.teams_enabled == 1 then -- enforce FFA-only label
   for each object with label "ffa_only" do
      current_object.delete()
   end
end

if game.teams_enabled == 0 then -- enforce team-only label
   for each object with label "team_only" do
      current_object.delete()
   end
end

on pregame: do
   game.symmetry = 1
   if game.teams_enabled == 1 then 
      game.symmetry = 0
   end
end

on init: do
   for each object with label "hh_drop_point" do
      current_object.timer[1] = 1
      current_object.timer[1].set_rate(-100%)
   end
end

for each player do -- manage loadout palettes
   if current_player.is_elite() then 
      current_player.set_loadout_palette(elite_tier_1)
   end
   if not current_player.is_elite() then 
      current_player.set_loadout_palette(spartan_tier_1)
   end
end

for each player do -- announce game start
   current_player.announce_start_timer.set_rate(-100%)
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      send_incident(headhunter_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

for each object with label "hh_drop_point" do
   if opt_drop_movement == -1 and not current_object.spawn_sequence < opt_drop_points then
      --
      -- If goal movement is disabled, then delete excess goal zones.
      --
      current_object.delete()
   end
   if opt_drop_movement != -1 then 
      current_object.team = neutral_team
   end
end

for each player do -- round card text
   if opt_headshots_only == 0 and game.score_to_win != 0 then 
      current_player.set_round_card_title("Capture skulls to earn points.\r\n%n points to win.", game.score_to_win)
   end
   if opt_headshots_only == 0 and game.score_to_win == 0 then 
      current_player.set_round_card_title("Capture skulls to earn points.")
   end
   if opt_headshots_only == 1 and game.score_to_win != 0 then 
      current_player.set_round_card_title("Capture skulls from headshots only.\r\n%n points to win.", game.score_to_win)
   end
   if opt_headshots_only == 1 and game.score_to_win == 0 then 
      current_player.set_round_card_title("Capture skulls from headshots only.")
   end
end

do -- player traits
   for each player do
      if current_player.carried_skulls > 0 then 
         current_player.apply_traits(skull_carrier_traits)
      end
   end
   for each player do
      if current_player.carried_skulls == 10 then 
         current_player.apply_traits(max_skull_carrier_traits)
      end
   end
end

on host migration: do
   for each player do
      global.object[0] = current_player.biped
      if global.object[0] == no_object then
         --
         -- If the player has no biped, then they are dead. They would have 
         -- been in the process of dropping (i.e. spawning) their skulls, 
         -- but the host migration would have interrupted this. Zeroing out 
         -- their carried skulls will pretty much cause the skulls that they 
         -- didn't get a chance to drop to disappear into the ether, but I'm 
         -- sure there's some edge-case that Bungie is preventing by doing 
         -- this.
         --
         current_player.carried_skulls = 0
      end
   end
end

for each player do -- some UI stuff, carried skulls dropping upon death, etc.
   alias current_biped = global.object[0]
   --
   current_player.skull_take_announce_cooldown.set_rate(-100%)
   ui_skull_count.set_text("SKULLS: %n", hud_player.carried_skulls)
   current_player.biped.set_shape(cylinder, 6, 4, 2) -- if a skull enters this shape, the player will pick it up
   current_biped = current_player.biped
   if current_player.killer_type_is(kill) then
      --
      -- When the player dies, we spawn skulls on their corpse. The amount 
      -- that we spawn depends on their carried skull count, so upon death 
      -- a player's skull count is temporarily incremented.
      --
      current_player.carried_skulls += 1
      if opt_headshots_only == 1 then 
         global.number[2] = current_player.try_get_death_damage_mod()
         if not global.number[2] == 5 then -- if not a headshot kill,
            current_player.carried_skulls -= 1 -- then never mind
         end
      end
   end
   --
   -- When a player dies, their "biped" property is cleared out. This has 
   -- two implications: if current_biped == no_object, then the player is 
   -- dead; and we need to store the player's current biped in a variable 
   -- when we do have access to it, so that we can spawn skulls from it 
   -- after the player dies.
   --
   if not current_biped == no_object then -- remember player's last biped, and handle their waypoint
      current_player.last_biped = current_biped
      current_player.biped.set_waypoint_icon(none)
      if current_player.carried_skulls > 0 then 
         current_player.biped.set_waypoint_icon(territory_a, current_player.carried_skulls)
         if current_player.carried_skulls > 9 then -- waypoint on players with 10 or more skulls
            current_player.biped.set_waypoint_icon(vip)
            current_player.biped.set_waypoint_priority(blink)
         end
      end
   end
   if not current_biped == no_object and current_player.carried_skulls > current_player.max_skulls then 
      --
      -- Manage the "Max Skulls" stat.
      --
      current_player.max_skulls = current_player.carried_skulls
   end
   --
   -- Spawn skulls on the player when they die:
   --
   if current_biped == no_object and current_player.carried_skulls > 0 and not current_player.last_biped == no_object then 
      --
      -- We'll spawn just one skull per script tick, decrementing the player's 
      -- carried skull count each time.
      --
      -- Megalo has no range loops (i.e. for i = 1, 5), and though the bytecode 
      -- allows for callable functions (which can be used to perform ranged 
      -- loops by way of recursion), the language that Bungie designed does not 
      -- offer that functionality. This means that Bungie can't just loop from 
      -- 0 to the player's carried skull count to spawn all the dropped skulls 
      -- all at once; the fact that the entire gametype script runs once per 
      -- frame basically "is" their only available looping mechanism.
      --
      -- This has some important implications: I'm not 100% sure that you can 
      -- enable instant respawn in Headhunter without breaking this here code. 
      -- If a player respawns too quickly, I'd expect them to retain some or 
      -- all of the skulls that they should've lost upon dying.
      --
      alias new_skull = global.object[1]
      --
      new_skull = no_object
      new_skull = current_player.last_biped.place_at_me(skull, none, suppress_effect | absolute_orientation, 0, 0, 4, none)
      new_skull.set_scale(200)
      new_skull.push_upward()
      new_skull.set_pickup_permissions(no_one)
      current_player.carried_skulls -= 1
   end
end

for each object with label all_skulls do -- handle skull pickup
   alias picker_upper = global.player[0]
   --
   picker_upper = no_player
   for each player do
      if picker_upper == no_player and current_player.biped.shape_contains(current_object) then 
         picker_upper = current_player
      end
   end
   if not picker_upper == no_player then 
      if picker_upper.carried_skulls < 10 then 
         picker_upper.carried_skulls += 1
      end
      current_object.delete()
      game.play_sound_for(picker_upper, timer_beep, true)
      if picker_upper.skull_take_announce_cooldown.is_zero() then 
         send_incident(skulls_taken, picker_upper, no_player)
         picker_upper.skull_take_announce_cooldown.reset()
      end
   end
end

for each object with label "hh_drop_point" do -- scoring
   if current_object.is_active == 1 then 
      alias is_instant_win = global.number[0]
      --
      is_instant_win = 0
      for each player do -- Handle Skullamanjaro.
         if  opt_instant_win == 1
         and current_player.carried_skulls == 10
         and current_object.shape_contains(current_player.biped)
         and not current_object.team.has_alliance_status(current_player.team, enemy) -- goals can be team-owned
         then 
            send_incident(skullamanjaro, current_player, all_players)
            send_incident(dlc_achieve_2, current_player, current_player, 67)
            if game.score_to_win != 0 then 
               current_player.score = game.score_to_win
               is_instant_win = 1
            end
            current_player.carried_skulls = 0
            if game.score_to_win == 0 then 
               current_player.score += 10
            end
         end
      end
      --
      -- Non-Skullamanjaro scoring uses two different triggers, so that we know what 
      -- kill feed message to display (single/plural).
      --
      for each player do -- Handle scoring a single skull.
         if  is_instant_win == 0
         and current_player.carried_skulls == 1
         and current_object.shape_contains(current_player.biped)
         and not current_object.team.has_alliance_status(current_player.team, enemy)
         then 
            game.show_message_to(current_player, announce_headhunter, "You scored one skull")
            send_incident(skulls_scored, current_player, no_player)
            current_player.score += current_player.carried_skulls
            current_player.carried_skulls = 0
         end
      end
      for each player do -- Handle scoring multiple skulls.
         if  is_instant_win == 0
         and current_player.carried_skulls > 0
         and current_object.shape_contains(current_player.biped)
         and not current_object.team.has_alliance_status(current_player.team, enemy)
         then 
            game.show_message_to(current_player, announce_headhunter, "You scored %n skulls", hud_player.carried_skulls)
            send_incident(skulls_scored, current_player, no_player)
            current_player.score += current_player.carried_skulls
            current_player.carried_skulls = 0
         end
      end
      --
      -- Delete skulls that land in an active goal zone:
      --
      alias current_zone = global.object[0]
      --
      current_zone = current_object
      for each object with label all_skulls do
         if current_zone.shape_contains(current_object) then 
            current_object.delete()
         end
      end
   end
end

do -- move drop points when the timer elapses
   drop_point_movement_timer.set_rate(-100%)
   if drop_point_movement_timer.is_zero() then 
      for each object with label "hh_drop_point" do
         if current_object.is_active == 1 then 
            current_object.activation_cooldown.reset()
            current_object.is_active = 0
         end
      end
      drop_point_movement_timer = opt_drop_movement
      game.show_message_to(all_players, announce_destination_moved, "Destination Moved")
   end
end

if opt_drop_movement != -1 then -- ensure minimum number of active goals (if movement is enabled)
   alias active_goal_count = global.number[1]
   --
   active_goal_count = 0
   for each object with label "hh_drop_point" do
      if current_object.is_active == 1 then 
         active_goal_count += 1
      end
   end
   if active_goal_count < opt_drop_points then 
      do
         global.object[0] = no_object
         global.object[0] = get_random_object("hh_drop_point", no_object)
         if global.object[0].is_active == 0 and global.object[0].activation_cooldown.is_zero() then 
            global.object[0].is_active = 1
         end
      end
   end
end
if opt_drop_movement == -1 then -- if movement is not enabled, set all goals on the map (that we haven't deleted) as active
   for each object with label "hh_drop_point" do
      current_object.is_active = 1
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each object with label "hh_drop_point" do -- manage drop points' activation cooldowns and visibility
   current_object.activation_cooldown.set_rate(-100%)
   current_object.set_waypoint_priority(high)
   --
   current_object.set_waypoint_visibility(no_one)
   current_object.set_shape_visibility(no_one)
   if current_object.is_active == 1 then 
      current_object.set_waypoint_visibility(everyone)
      current_object.set_shape_visibility(everyone)
      --
      -- If the goal is about to move (and if goal movement is even enabled), 
      -- blink its waypoint:
      --
      if drop_point_movement_timer < 6 then 
         current_object.set_waypoint_priority(blink)
         if opt_drop_movement == -1 then 
            current_object.set_waypoint_priority(high)
         end
      end
   end
end

do -- cap the number of skulls on the map
   alias skulls_on_map = global.number[0]
   --
   skulls_on_map = 0
   for each object with label all_skulls do
      skulls_on_map += 1
      if skulls_on_map > 100 then 
         current_object.delete()
      end
   end
end
for each object with label all_skulls do -- delete out-of-bounds skulls
   if current_object.is_out_of_bounds() then 
      current_object.delete()
   end
end
