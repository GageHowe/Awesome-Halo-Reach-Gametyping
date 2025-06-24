
alias opt_capture_time    = script_option[0]
alias opt_reinforce_time  = script_option[1]
alias opt_deadline_t2     = script_option[2]
alias opt_deadline_t3     = script_option[3]
alias opt_score_limit_t2  = script_option[4]
alias opt_score_limit_t3  = script_option[5]
alias opt_vehicle_limit   = script_option[6] -- value is 15; cannot be changed without modding
alias opt_kill_points     = script_option[7]
alias opt_suicide_points  = script_option[8]
alias opt_betrayal_points = script_option[9]
alias opt_headshot_bonus  = script_option[10]
alias opt_pummel_bonus    = script_option[11]
alias opt_splatter_bonus  = script_option[12]
alias opt_sticky_bonus    = script_option[13]
alias opt_spree_bonus     = script_option[14]

alias rating_stat = player.script_stat[0]

-- Unnamed Forge labels:
alias all_ghosts    = 7
alias all_warthogs  = 8
alias all_banshees  = 9
alias all_wraiths   = 10
alias all_scorpions = 11

alias team_unsc = team[0]
alias team_covy = team[1]

alias announced_game_start = player.number[2]
alias announce_start_timer = player.timer[0]

alias current_tier        = global.number[0]
alias temp_int_00         = global.number[1] -- unused
alias temp_int_01         = global.number[2]
alias temp_int_02         = global.number[3]
alias temp_int_03         = global.number[4]
alias temp_int_04         = global.number[5]
alias current_drop_zone   = global.object[0]
alias last_drop_zone      = global.object[1] -- the last drop zone to be captured
alias temp_tem_00         = global.team[0] -- unused
alias temp_tem_01         = global.team[1]
alias temp_tem_02         = global.team[2] -- unused
alias time_to_next_zone   = global.timer[0]
alias is_contested        = object.number[2] -- for drop zones
alias budget_cost         = object.number[3] -- set, but not read; matches Vehicle Limit trigger
alias capture_timer       = object.timer[0]
alias reinforce_timer     = object.timer[1]
alias invincibility_timer = object.timer[2] -- for created vehicles; prevents them from being destroyed immediately
alias fireteam            = player.number[1]

declare current_tier with network priority low = 1
declare temp_int_00  with network priority local
declare temp_int_01  with network priority local
declare temp_int_02  with network priority local
declare temp_int_03  with network priority local
declare temp_int_04  with network priority local
declare current_drop_zone with network priority low
declare last_drop_zone    with network priority low
declare global.object[2] with network priority local -- temporary
declare global.object[3] with network priority local -- temporary
declare global.player[0] with network priority local -- temporary
declare temp_tem_00 with network priority low
declare temp_tem_01 with network priority local
declare temp_tem_02 with network priority local
declare time_to_next_zone = 20
declare player.number[0] with network priority local -- unused
declare player.number[1] with network priority low   -- unused
declare player.announced_game_start with network priority low
declare player.announce_start_timer = 5
declare object.number[0] with network priority local -- unused
declare object.number[1] with network priority local -- unused
declare object.is_contested with network priority low
declare object.budget_cost  with network priority local
declare object.capture_timer       = opt_capture_time
declare object.reinforce_timer     = opt_reinforce_time
declare object.invincibility_timer = 5

on pregame: do
   game.symmetry = 1
end

on init: do
end

for each player do -- why is this its own trigger? lol
   current_player.announce_start_timer.set_rate(-100%)
end
for each player do -- round card and announce game start
   if current_player.announced_game_start == 0 then 
      if game.score_to_win != 0 then 
         current_player.set_round_card_title("Kill players on the enemy team.\r\n%n points to win.", game.score_to_win)
      end
      if game.score_to_win == 0 then 
         current_player.set_round_card_title("Kill players on the enemy team.")
      end
      if current_player.announce_start_timer.is_zero() then 
         send_incident(invasion_slayer_start, current_player, no_player)
         current_player.announced_game_start = 1
      end
   end
end

for each object with label "inv_slayer_res_zone" do -- limit available spawn zones by fireteam
   alias current_zone = global.object[2]
   --
   current_object.set_spawn_location_permissions(allies)
   current_zone = current_object
   for each player do
      current_player.fireteam = current_player.get_fireteam()
      if current_zone.spawn_sequence == current_player.fireteam then 
         current_player.set_primary_respawn_object(current_zone)
      end
   end
end

for each team do -- enable co-op spawning for all teams
   if current_team.has_any_players() then 
      current_team.set_co_op_spawning(true)
   end
end

for each player do -- give players waypoints on their allies... and show Arena rating in the scoreboard?
   current_player.biped.set_waypoint_visibility(allies)
   current_player.rating_stat = current_player.rating
end

do -- handle loadout palettes and advancing to tiers 2 and 3
   alias deadline_t2_s = temp_int_02
   alias deadline_t3_s = temp_int_03
   --
   -- Tier advancement trigger: time limit
   --
   deadline_t2_s = opt_deadline_t2
   deadline_t3_s = opt_deadline_t3
   deadline_t2_s *= 60 -- minutes -> seconds
   deadline_t3_s *= 60
   if game.round_timer < deadline_t3_s and current_tier < 3 then 
      current_tier = 3
      game.play_sound_for(all_players, unused_87, true)
      for each player do
         current_player.set_round_card_title("TIER 3 UNLOCKED: New Loadouts\r\nand Reinforcements Available")
      end
   end
   if game.round_timer < deadline_t2_s and current_tier < 2 then 
      current_tier = 2
      game.play_sound_for(all_players, unused_87, true)
      for each player do
         current_player.set_round_card_title("TIER 2 UNLOCKED: New Loadouts\r\nand Reinforcements Available")
      end
   end
   --
   -- Tier advancement trigger: score limit
   --
   for each team do
      if current_team == team_covy and not current_team.score < opt_score_limit_t2 and current_tier < 2 then 
         current_tier = 2
         game.play_sound_for(all_players, inv_cue_covenant_win_1, true)
         for each player do
            current_player.set_round_card_title("TIER 2 UNLOCKED: New Loadouts\r\nand Reinforcements Available")
         end
      end
   end
   for each team do
      if current_team == team_unsc and not current_team.score < opt_score_limit_t2 and current_tier < 2 then 
         current_tier = 2
         game.play_sound_for(all_players, inv_cue_spartan_win_1, true)
         for each player do
            current_player.set_round_card_title("TIER 2 UNLOCKED: New Loadouts\r\nand Reinforcements Available")
         end
      end
   end
   for each team do
      if current_team == team_covy and not current_team.score < opt_score_limit_t3 and current_tier < 3 then 
         current_tier = 3
         game.play_sound_for(all_players, inv_cue_covenant_win_2, true)
         for each player do
            current_player.set_round_card_title("TIER 3 UNLOCKED: New Loadouts\r\nand Reinforcements Available")
         end
      end
   end
   for each team do
      if current_team == team_unsc and not current_team.score < opt_score_limit_t3 and current_tier < 3 then 
         current_tier = 3
         game.play_sound_for(all_players, inv_cue_spartan_win_2, true)
         for each player do
            current_player.set_round_card_title("TIER 3 UNLOCKED: New Loadouts\r\nand Reinforcements Available")
         end
      end
   end
   --
   -- Loadout palettes:
   --
   for each player do
      if current_tier == 1 then 
         if current_player.team == team_unsc then 
            current_player.set_loadout_palette(spartan_tier_1)
         end
         if current_player.team == team_covy then 
            current_player.set_loadout_palette(elite_tier_1)
         end
      end
   end
   for each player do
      if current_tier == 2 then 
         if current_player.team == team_unsc then 
            current_player.set_loadout_palette(spartan_tier_2)
         end
         if current_player.team == team_covy then 
            current_player.set_loadout_palette(elite_tier_2)
         end
      end
   end
   for each player do
      if current_tier == 3 then 
         if current_player.team == team_unsc then 
            current_player.set_loadout_palette(spartan_tier_3)
         end
         if current_player.team == team_covy then 
            current_player.set_loadout_palette(elite_tier_3)
         end
      end
   end
end

for each player do -- kill points trigger
   alias killer = global.player[0]
   --
   temp_int_02 = 0
   if current_player.killer_type_is(kill) then 
      killer = current_player.try_get_killer()
      killer.score += opt_kill_points
      temp_int_01 = current_player.try_get_death_damage_mod()
      temp_int_02 = killer.get_spree_count()
      do
         temp_int_02 %= 5
         if temp_int_02 == 0 then 
            killer.score += opt_spree_bonus
         end
      end
      if temp_int_01 == enums.damage_reporting_modifier.pummel then 
         killer.score += opt_pummel_bonus
      end
      if temp_int_01 == enums.damage_reporting_modifier.splatter then 
         killer.score += opt_splatter_bonus
      end
      if temp_int_01 == enums.damage_reporting_modifier.sticky then 
         killer.score += opt_sticky_bonus
      end
      if temp_int_01 == enums.damage_reporting_modifier.headshot then 
         killer.score += opt_headshot_bonus
      end
   end
end

for each player do -- award suicide points
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) and not current_player.killer_type_is(kill) and not current_player.killer_type_is(betrayal) then 
      current_player.score += opt_suicide_points
   end
end

for each player do -- award betrayal points
   alias killer = global.player[0]
   if current_player.killer_type_is(betrayal) then 
      killer = current_player.try_get_killer()
      killer.score += opt_betrayal_points
   end
end

--
-- This trigger will run at match start, and between a territory being captured and 
-- the territory moving.
--
if current_drop_zone == no_object then -- pick the next drop zone
   alias budget = temp_int_02
   alias health = temp_int_03
   --
   budget = 0
   for each object with label all_ghosts do
      health = 0
      health = current_object.health
      if health > 0 then -- only track vehicles that are alive
         budget += 4
      end
   end
   for each object with label all_warthogs do
      health = 0
      health = current_object.health
      if health > 0 then 
         budget += 8
      end
   end
   for each object with label all_banshees do
      health = 0
      health = current_object.health
      if health > 0 then 
         budget += 3
      end
   end
   for each object with label all_wraiths do
      health = 0
      health = current_object.health
      if health > 0 then 
         budget += 6
      end
   end
   for each object with label all_scorpions do
      health = 0
      health = current_object.health
      if health > 0 then 
         budget += 8
      end
   end
   if budget < opt_vehicle_limit then -- opt_vehicle_limit == 15; cannot be changed without modding
      time_to_next_zone.set_rate(-100%)
      if time_to_next_zone.is_zero() then 
         current_drop_zone = get_random_object("inv_slayer_drop", last_drop_zone)
         game.play_sound_for(all_players, announce_destination_moved, false)
         time_to_next_zone.reset() -- NOTE: when the timer reached zero, its rate was implicitly changed to 0%
         current_drop_zone.capture_timer.reset()
         current_drop_zone.team = neutral_team
      end
   end
end

if not current_drop_zone == no_object then 
   for each object with label "inv_slayer_drop" do
      if current_object == current_drop_zone then
         alias unsc_inside = temp_int_02
         alias covy_inside = temp_int_03
         alias all_inside  = temp_int_04 -- not actually used
         alias capturing_team = temp_tem_01
         --
         global.object[2] = current_drop_zone
         global.object[2].set_waypoint_icon(ordnance)
         global.object[2].set_waypoint_timer(object.capture_timer)
         global.object[2].set_waypoint_priority(high)
         global.object[2].set_waypoint_visibility(everyone)
         global.object[2].set_progress_bar(object.capture_timer, no_one)
         global.object[2].capture_timer.set_rate(100%) -- capture timer refills when the territory is empty
         global.object[2].is_contested = 0
         global.object[2].set_shape_visibility(everyone)
         unsc_inside = 0
         covy_inside = 0
         all_inside  = 0
         capturing_team = no_team
         temp_tem_02    = no_team
         current_drop_zone.team = neutral_team
         for each player do
            alias current_vehicle = global.object[3]
            if global.object[2].shape_contains(current_player.biped) then 
               current_vehicle = no_object
               current_vehicle = current_player.try_get_vehicle()
               if current_vehicle == no_object then -- players in vehicles cannot capture or contest territories
                  global.object[2].capture_timer.set_rate(-100%) -- capture timer counts down when the territory is being captured
                  global.object[2].set_waypoint_priority(blink)
                  all_inside  += 1
                  unsc_inside += 1
                  capturing_team = team_unsc
                  if current_player.team == team_covy then 
                     unsc_inside -= 1
                     covy_inside += 1
                     capturing_team = team_covy
                  end
               end
            end
         end
         if covy_inside > 0 and unsc_inside > 0 then -- territory contested
            capturing_team = no_team
            global.object[2].is_contested = 1
            global.object[2].capture_timer.set_rate(100%) -- capture timer refills when the territory is contested
         end
         if not capturing_team == no_team then 
            temp_tem_00 = capturing_team
            current_drop_zone.team = capturing_team
         end
         global.object[2].set_progress_bar(object.capture_timer, everyone)
         if global.object[2].capture_timer.is_zero() and not capturing_team == no_team then
            global.object[2].is_contested = 0
            game.show_message_to(capturing_team, announce_territories_captured, "Territory Captured")
            global.object[2].team = capturing_team
            global.object[2].reinforce_timer.set_rate(-100%)
            global.object[2].set_waypoint_priority(high)
            global.object[2].set_waypoint_timer(object.reinforce_timer)
            global.object[2].set_waypoint_icon(supply)
            global.object[2].set_shape_visibility(no_one)
            global.object[2].set_progress_bar(object.capture_timer, no_one)
            --
            -- When a zone is captured, clear (current_drop_zone) so that the timer for 
            -- moving to the next zone will start. Before clearing it, store it into 
            -- (last_drop_zone) so that when we pick a new drop zone at random, we don't 
            -- end up using this zone twice in a row.
            --
            last_drop_zone    = current_drop_zone
            current_drop_zone = no_object
         end
      end
   end
end

if current_tier == 1 then -- Tier 1 Reinforements: power weapons
   for each object with label "inv_slayer_drop" do
      if current_object.reinforce_timer.is_zero() and current_object.team == team_unsc then 
         game.play_sound_for(all_players, firefight_lives_added, false)
         global.object[2] = no_object
         global.object[2] = current_object.place_at_me(rocket_launcher, none, none, 0, 0, 2, none)
         global.object[2] = current_object.place_at_me(sniper_rifle,    none, none, 0, 0, 2, none)
         game.show_message_to(current_object.team, none, "Weapons Delivered")
         current_object.reinforce_timer.reset()
         current_object.team = neutral_team
         current_object.set_waypoint_visibility(no_one)
      end
   end
   for each object with label "inv_slayer_drop" do
      if current_object.reinforce_timer.is_zero() and current_object.team == team_covy then 
         game.play_sound_for(all_players, firefight_lives_added, false)
         global.object[2] = no_object
         global.object[2] = current_object.place_at_me(plasma_launcher, none, none, 0, 0, 3, none)
         global.object[2] = current_object.place_at_me(focus_rifle,     none, none, 0, 0, 3, none)
         game.show_message_to(current_object.team, none, "Weapons Delivered")
         current_object.reinforce_timer.reset()
         current_object.team = neutral_team
         current_object.set_waypoint_visibility(no_one)
      end
   end
end

if current_tier == 2 then -- Tier 2 Reinforcements: Warthog / Ghost / Banshee / power weapons
   alias outcome = temp_int_02
   --
   for each object with label "inv_slayer_drop" do
      if current_object.reinforce_timer.is_zero() and current_object.team == team_unsc then 
         game.play_sound_for(all_players, firefight_lives_added, false)
         outcome = 0
         outcome = rand(3)
         if outcome < 2 then -- 66.6% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(warthog, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 8
            game.show_message_to(current_object.team, none, "Warthog Delivered")
         end
         if outcome == 2 then -- 33.3% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(rocket_launcher, none, none, 0, 0, 2, none)
            global.object[2] = current_object.place_at_me(spartan_laser,   none, none, 0, 0, 2, none)
            game.show_message_to(current_object.team, none, "Weapons Delivered")
         end
         current_object.reinforce_timer.reset()
         current_object.team = neutral_team
         current_object.set_waypoint_visibility(no_one)
      end
   end
   for each object with label "inv_slayer_drop" do
      if current_object.reinforce_timer.is_zero() and current_object.team == team_covy then 
         game.play_sound_for(all_players, firefight_lives_added, false)
         outcome = 0
         outcome = rand(4)
         if outcome < 2 then -- 50% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(banshee, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 3
            game.show_message_to(current_object.team, none, "Banshee Delivered")
         end
         if outcome == 2 then -- 25% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(ghost, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 4
            game.show_message_to(current_object.team, none, "Ghost Delivered")
         end
         if outcome == 3 then -- 25% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(plasma_launcher, none, none, 0, 0, 3, none)
            global.object[2] = current_object.place_at_me(fuel_rod_gun,    none, none, 0, 0, 3, none)
            game.show_message_to(current_object.team, none, "Weapons Delivered")
         end
         current_object.reinforce_timer.reset()
         current_object.team = neutral_team
         current_object.set_waypoint_visibility(no_one)
      end
   end
end

if current_tier == 3 then -- Tier 3 Reinforcements: Scorpion / Warthog / Wraith / Ghost / Banshee / power weapons
   alias outcome = temp_int_02
   --
   for each object with label "inv_slayer_drop" do
      if current_object.reinforce_timer.is_zero() and current_object.team == team_unsc then 
         game.play_sound_for(all_players, firefight_lives_added, false)
         outcome = 0
         outcome = rand(4)
         if outcome < 2 then -- 50% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(scorpion, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 8
            game.show_message_to(current_object.team, none, "Scorpion Delivered")
         end
         if outcome == 2 then -- 25% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(warthog, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 8
            game.show_message_to(current_object.team, none, "Warthog Delivered")
         end
         if outcome == 3 then -- 25% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(rocket_launcher, none, none, 0, 0, 2, none)
            global.object[2] = current_object.place_at_me(sniper_rifle,    none, none, 0, 0, 2, none)
            global.object[2] = current_object.place_at_me(spartan_laser,   none, none, 0, 0, 2, none)
            game.show_message_to(current_object.team, none, "Weapons Delivered")
         end
         current_object.reinforce_timer.reset()
         current_object.team = neutral_team
         current_object.set_waypoint_visibility(no_one)
      end
   end
   for each object with label "inv_slayer_drop" do
      if current_object.reinforce_timer.is_zero() and current_object.team == team_covy then 
         game.play_sound_for(all_players, firefight_lives_added, false)
         outcome = 0
         outcome = rand(5)
         if outcome < 2 then -- 40% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(wraith, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 6
            game.show_message_to(current_object.team, none, "Wraith Delivered")
         end
         if outcome == 2 then -- 20% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(ghost, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 4
            game.show_message_to(current_object.team, none, "Ghost Delivered")
         end
         if outcome == 3 then -- 20% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(banshee, "created_vehicles", none, 0, 0, 15, none)
            global.object[2].budget_cost = 3
            game.show_message_to(current_object.team, none, "Banshee Delivered")
         end
         if outcome == 4 then -- 20% chance
            global.object[2] = no_object
            global.object[2] = current_object.place_at_me(focus_rifle,     none, none, 0, 0, 2, none)
            global.object[2] = current_object.place_at_me(plasma_launcher, none, none, 0, 0, 2, none)
            global.object[2] = current_object.place_at_me(fuel_rod_gun,    none, none, 0, 0, 2, none)
            game.show_message_to(current_object.team, none, "Weapons Delivered")
         end
         current_object.reinforce_timer.reset()
         current_object.team = neutral_team
         current_object.set_waypoint_visibility(no_one)
      end
   end
end

if game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each object with label "inv_platform" do
   current_object.set_device_position(0)
end

for each object with label "created_vehicles" do -- created vehicles should be briefly invulnerable after spawning
   current_object.set_invincibility(1)
   current_object.invincibility_timer.set_rate(-100%)
   if current_object.invincibility_timer.is_zero() then 
      current_object.set_invincibility(0)
   end
end

for each object with label "none" do
   current_object.delete()
end
