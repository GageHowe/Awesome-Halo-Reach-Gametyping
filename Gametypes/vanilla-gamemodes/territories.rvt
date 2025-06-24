
alias opt_symmetry          = script_option[0]
alias opt_terr_count        = script_option[1]
alias opt_capture_time      = script_option[2]
alias opt_contested_scoring = script_option[3]
alias opt_capture_lock      = script_option[4]

alias in_terr_traits = script_traits[0]

-- Unnamed Forge labels:
alias all_jetpacks    = 0
alias all_armor_locks = 1
alias all_anti_spawn_zones = 5

alias captures = player.script_stat[0]

alias announced_game_start = player.number[2]
alias announce_start_timer = player.timer[1]

alias sudden_death_enabled   = global.number[0]
alias announced_sudden_death = global.number[1]
alias temp_int_00    = global.number[2]
alias score_on_timer = global.number[4] -- if 1, then you earn points per second for territories owned
alias temp_int_01    = global.number[3] -- unused
alias temp_int_02    = global.number[5]
alias temp_plr_00    = global.player[0]
alias scoring_interval = global.timer[0] -- used to award points per second
alias terr_contest_announce_cooldown = global.timer[1]
alias allies_inside  = object.number[0]
alias enemies_inside = object.number[1]
alias is_preplaced   = object.number[2] -- used to delete pre-placed Weak Anti Respawn Zones if they are not inside of a Territory
alias is_locked      = object.number[3]
alias is_contested   = object.number[4]
alias announced_is_contested = object.number[5] -- did we announce that this specific territory is contested?
alias decor_flag   = object.object[0]
alias being_captured_by = object.team[0]
alias owner             = object.team[1]
alias relevant_cap_time = object.timer[0] -- on the territory
alias cap_timer_offense = object.timer[0] -- on the flag; capture timer for team 1
alias cap_timer_defense = object.timer[1] -- on the flag; capture timer for team 0 (yes, Bungie mismatched indices)
alias cap_timer_team_02 = object.timer[2] -- on the flag
alias cap_timer_team_03 = object.timer[3] -- on the flag
alias ach_top_shot_count        = player.number[0]
alias ach_license_to_kill_count = player.number[1]
alias ach_paper_beats_rock_vuln_timer = player.timer[0]
alias owned_terr_count     = team.number[0]
alias contested_terr_count = team.number[1]
alias announced_30s_win    = team.number[2]
alias announced_60s_win    = team.number[3]

declare sudden_death_enabled   with network priority local
declare announced_sudden_death with network priority local
declare temp_int_00    with network priority local
declare temp_int_01    with network priority local -- unused
declare score_on_timer with network priority low = 1
declare temp_int_02    with network priority local
declare global.object[0] with network priority local
declare global.object[1] with network priority local
declare global.object[2] with network priority local
declare temp_plr_00 with network priority local
declare global.team[0] with network priority low -- written to, but never read
declare global.team[1] with network priority local
declare global.team[2] with network priority local
declare scoring_interval = 1
declare terr_contest_announce_cooldown = 3
declare player.ach_top_shot_count              with network priority low
declare player.ach_paper_beats_rock_vuln_timer with network priority low
declare player.announced_game_start with network priority low
declare player.announce_start_timer = 5
declare object.allies_inside  with network priority local
declare object.enemies_inside with network priority local
declare object.is_preplaced   with network priority local = 1
declare object.is_locked      with network priority low
declare object.is_contested   with network priority low
declare object.announced_is_contested with network priority low
declare object.decor_flag        with network priority low
declare object.being_captured_by with network priority low
declare object.owner             with network priority low = neutral_team
declare object.cap_timer_offense = opt_capture_time
declare object.cap_timer_defense = opt_capture_time
declare object.cap_timer_team_02 = opt_capture_time
declare object.cap_timer_team_03 = opt_capture_time
declare team.owned_terr_count     with network priority low
declare team.contested_terr_count with network priority low
declare team.announced_30s_win    with network priority low
declare team.announced_60s_win    with network priority low

do
   sudden_death_enabled = 0
end

-- Start of DLC achievement boilerplate

for each player do -- award Dive Bomber achievement as appropriate
   alias killer = temp_plr_00
   if current_player.killer_type_is(kill) then 
      killer = no_player
      killer = current_player.try_get_killer()
      temp_int_00 = 0
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.assassination then 
         global.object[0] = no_object
         global.object[0] = killer.try_get_armor_ability()
         if global.object[0].has_forge_label(all_jetpacks) and global.object[0].is_in_use() then 
            send_incident(dlc_achieve_2, killer, killer, 65)
         end
      end
   end
end

for each player do -- award From Hell's Heart achievement as appropriate
   if current_player.killer_type_is(kill) then 
      temp_int_00 = 0
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.sticky then 
         temp_plr_00 = no_player
         temp_plr_00 = current_player.try_get_killer()
         if temp_plr_00.killer_type_is(suicide) then 
            send_incident(dlc_achieve_2, current_player, current_player, 68)
         end
      end
   end
end

for each player do -- manage and award Top Shot achievement as appropriate
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.ach_top_shot_count = 0
      if current_player.killer_type_is(kill) then 
         temp_plr_00 = no_player
         temp_plr_00 = current_player.try_get_killer()
         temp_int_00 = 0
         temp_int_00 = current_player.try_get_death_damage_mod()
         if temp_int_00 != enums.damage_reporting_modifier.headshot then 
            temp_plr_00.ach_top_shot_count = 0
         end
         if temp_int_00 == enums.damage_reporting_modifier.headshot then 
            temp_plr_00.ach_top_shot_count += 1
         end
         if temp_plr_00.ach_top_shot_count > 2 then 
            send_incident(dlc_achieve_2, temp_plr_00, temp_plr_00, 62)
         end
      end
   end
end

for each player do -- manage and award License To Kill achievement as appropriate
   if current_player.killer_type_is(kill) then 
      temp_plr_00 = no_player
      temp_plr_00 = current_player.try_get_killer()
      temp_int_00 = 0
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.splatter then 
         temp_plr_00.ach_license_to_kill_count += 1
      end
      if temp_plr_00.ach_license_to_kill_count > 4 then 
         send_incident(dlc_achieve_2, temp_plr_00, temp_plr_00, 66)
      end
   end
end

for each player do -- manage timing for the Paper Beats Rock achievement
   global.object[0] = no_object
   global.object[0] = current_player.try_get_armor_ability()
   if global.object[0].has_forge_label(all_armor_locks) and global.object[0].is_in_use() then 
      current_player.ach_paper_beats_rock_vuln_timer = 3
      current_player.ach_paper_beats_rock_vuln_timer.set_rate(-100%)
   end
end
for each player do -- award Paper Beats Rock achievement as appropriate
   if current_player.killer_type_is(kill) and not current_player.ach_paper_beats_rock_vuln_timer.is_zero() then 
      temp_int_00 = 0
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.assassination then 
         temp_plr_00 = no_player
         temp_plr_00 = current_player.try_get_killer()
         send_incident(dlc_achieve_2, temp_plr_00, temp_plr_00, 60)
      end
   end
end

-- End of DLC achievement boilerplate

on pregame: do
   game.symmetry = 1
   if opt_symmetry == 0 then 
      game.symmetry = 0
   end
end

do
   script_widget[0].set_text("Territory Contested")
   for each object with label "terr_object" do -- delete extra territories
      if current_object.spawn_sequence > opt_terr_count or current_object.spawn_sequence == 0 then 
         current_object.delete()
      end
   end
end

for each object with label "terr_object" do -- give all territories a decorative flag and a weak-anti respawn zone
   alias current_decor_flag = global.object[0]
   --
   if current_object.decor_flag == no_object then
      --
      -- If the territory doesn't have a decorative flag, then we'll also assume 
      -- that it's just spawned and needs to be appropriately configured.
      --
      if current_object.is_locked == 0 then 
         current_object.team = neutral_team
         if opt_symmetry == 0 then -- asymmetric i.e. attack/defend
            current_object.team  = team[0]
            current_object.owner = team[0]
         end
         current_object.set_waypoint_visibility(everyone)
         current_object.set_waypoint_icon(territory_a, current_object.spawn_sequence)
         current_object.set_shape_visibility(everyone)
      end
      --
      current_object.decor_flag = current_object.place_at_me(flag, none, never_garbage_collect, 0, 0, 2, none)
      current_object.decor_flag.set_pickup_permissions(no_one)
      current_decor_flag = current_object.decor_flag
      current_decor_flag.team = current_object.team
      if current_object.is_of_type(flag_stand) or current_object.is_of_type(capture_plate) then 
         current_object.decor_flag.attach_to(current_object, 0, 0, 3, absolute)
      end
      --
      global.object[1] = no_object
      global.object[1] = current_object.place_at_me(respawn_zone_weak_anti, none, never_garbage_collect, 0, 0, 0, none)
      global.object[1].team = neutral_team
      global.object[1].set_shape_visibility(no_one)
   end
end

for each player do -- loadout palettes
   if current_player.is_elite() then 
      current_player.set_loadout_palette(elite_tier_1)
   end
   if not current_player.is_elite() then 
      current_player.set_loadout_palette(spartan_tier_1)
   end
end

for each player do -- round card
   current_player.announce_start_timer.set_rate(-100%)
   if opt_symmetry == 1 and opt_capture_lock == 0 and game.score_to_win != 0 then 
      current_player.set_round_card_title("Capture the Territories to earn points.\r\n%n points to win.", game.score_to_win)
   end
   if opt_symmetry == 1 and opt_capture_lock == 0 and game.score_to_win == 0 then 
      current_player.set_round_card_title("Capture the Territories to earn points.")
   end
   if opt_symmetry == 1 and opt_capture_lock == 1 then 
      current_player.set_round_card_title("Capture the Territories to earn points.\r\n%n points to win.", opt_terr_count)
   end
   if opt_symmetry == 0 and current_player.team == team[0] then 
      current_player.set_round_card_title("Defend your Territories.\r\n%n points to win.", opt_terr_count)
      current_player.set_round_card_text("Defense")
      current_player.set_round_card_icon(defend)
   end
   if opt_symmetry == 0 and current_player.team == team[1] then 
      current_player.set_round_card_title("Steal enemy Territories for points.\r\n%n points to win.", opt_terr_count)
      current_player.set_round_card_text("Offense")
      current_player.set_round_card_icon(attack)
   end
end

for each player do -- announce game start
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      send_incident(terr_game_start, current_player, no_player)
      current_player.announced_game_start = 1
      if opt_symmetry == 0 and current_player.team == team[1] then 
         send_incident(team_offense, current_player, no_player)
      end
      if opt_symmetry == 0 and current_player.team == team[0] then 
         send_incident(team_defense, current_player, no_player)
      end
   end
end

on host migration: do
   for each object with label "terr_object" do -- delete excess territories
      if current_object.spawn_sequence > opt_terr_count then 
         current_object.delete()
      end
   end
end

for each object with label all_anti_spawn_zones do -- delete pre-placed Weak Anti Respawn Zones unless they are inside of a territory
   alias current_zone      = global.object[0]
   alias current_territory = global.object[1]
   --
   current_zone = current_object
   for each object with label "terr_object" do
      current_territory = current_object
      if current_territory.shape_contains(current_zone) then 
         current_zone.is_preplaced = 0
      end
   end
   if current_zone.is_preplaced == 1 then -- variable is initialized to 1
      current_zone.delete()
   end
end

do
   score_on_timer = 1
   if opt_symmetry == 1 and opt_capture_lock == 0 then 
      score_on_timer = 0
   end
end

for each object with label "terr_object" do -- track who is capturing a territory, and how many enemies and allies are inside
   alias current_territory    = global.object[0]
   alias player_on_foot_count = temp_int_00
   --
   current_territory = current_object
   current_territory.set_waypoint_priority(normal)
   current_territory.is_contested = 0
   current_territory.allies_inside  = 0
   current_territory.enemies_inside = 0
   current_territory.team[0] = no_team
   player_on_foot_count = 0
   for each player do
      script_widget[0].set_visibility(current_player, false)
      if current_territory.shape_contains(current_player.biped) then 
         if current_object.is_locked == 0 then 
            current_player.apply_traits(in_terr_traits)
         end
         global.object[1] = no_object
         global.object[1] = current_player.try_get_vehicle()
         if global.object[1] == no_object then -- players in a vehicle can't take territories
            player_on_foot_count += 1
            current_territory.enemies_inside += 1
            if player_on_foot_count == 1 then 
               current_territory.being_captured_by = current_player.team
            end
            if current_player.team == current_territory.owner and opt_symmetry == 0 or opt_capture_lock == 0 then 
               current_territory.enemies_inside -= 1
               current_territory.allies_inside  += 1
            end
            if current_player.team != current_territory.being_captured_by then 
               current_territory.being_captured_by = no_team
               current_territory.is_contested = 1
            end
         end
      end
   end
   if not current_territory.being_captured_by == no_team then 
      global.team[0] = current_territory.being_captured_by -- variable isn't used anywhere
   end
   if current_territory.enemies_inside <= 0 and current_territory.allies_inside <= 0 then 
      current_territory.announced_is_contested = 0
   end
end

for each object with label "terr_object" do -- manage territory progress bar visibility and capture timer rates
   alias current_territory = global.object[0]
   alias current_flag      = global.object[1]
   --
   current_territory = current_object
   current_flag      = current_territory.decor_flag
   current_object.set_progress_bar(object.relevant_cap_time, enemies)
   if current_territory.owner == neutral_team then 
      current_object.set_progress_bar(object.relevant_cap_time, everyone)
   end
   if opt_symmetry == 1 and current_territory.is_locked == 0 then 
      current_territory.team = current_territory.owner
   end
   current_territory.relevant_cap_time.set_rate(100%)
   current_flag.cap_timer_offense.set_rate(100%)
   current_flag.cap_timer_defense.set_rate(100%)
   current_flag.cap_timer_team_02.set_rate(100%)
   current_flag.cap_timer_team_03.set_rate(100%)
end

for each object with label "terr_object" do -- manage territory capture timers
   --
   -- A territory has a single timer representing the relevant capture time, and 
   -- the territory's flag has four capture timers: one for each team.
   --
   alias current_territory = global.object[0]
   alias current_flag      = global.object[1]
   alias total_inside      = temp_int_00 -- total number of players inside (current_territory); is not used
   --
   current_territory = current_object
   total_inside      = current_territory.allies_inside
   current_flag      = current_territory.decor_flag
   total_inside     += current_territory.enemies_inside
   if current_flag != no_object then 
      if current_territory.enemies_inside > 0 and current_territory.is_locked != 1 then
         current_territory.is_contested = 1
         current_territory.set_waypoint_priority(blink)
         if current_territory.being_captured_by == team[1] then 
            do
               current_flag.cap_timer_offense.set_rate(-100%)
               if current_territory.enemies_inside > 1 then 
                  current_flag.cap_timer_offense.set_rate(-200%)
                  if current_territory.enemies_inside > 2 then 
                     current_flag.cap_timer_offense.set_rate(-400%)
                  end
               end
            end
            current_territory.relevant_cap_time = current_flag.cap_timer_offense
         end
         if current_territory.being_captured_by == team[0] then 
            do
               current_flag.cap_timer_defense.set_rate(-100%)
               if current_territory.enemies_inside > 1 then 
                  current_flag.cap_timer_defense.set_rate(-200%)
                  if current_territory.enemies_inside > 2 then 
                     current_flag.cap_timer_defense.set_rate(-400%)
                  end
               end
            end
            current_territory.relevant_cap_time = current_flag.cap_timer_defense
         end
         if current_territory.being_captured_by == team[2] then 
            do
               current_flag.cap_timer_team_02.set_rate(-100%)
               if current_territory.enemies_inside > 1 then 
                  current_flag.cap_timer_team_02.set_rate(-200%)
                  if current_territory.enemies_inside > 2 then 
                     current_flag.cap_timer_team_02.set_rate(-400%)
                  end
               end
            end
            current_territory.relevant_cap_time = current_flag.cap_timer_team_02
         end
         if current_territory.being_captured_by == team[3] then 
            do
               current_flag.cap_timer_team_03.set_rate(-100%)
               if current_territory.enemies_inside > 1 then 
                  current_flag.cap_timer_team_03.set_rate(-200%)
                  if current_territory.enemies_inside > 2 then 
                     current_flag.cap_timer_team_03.set_rate(-400%)
                  end
               end
            end
            current_territory.relevant_cap_time = current_flag.cap_timer_team_03
         end
         if current_territory.being_captured_by == no_team then
            --
            -- The territory isn't being captured. We already checked earlier and 
            -- saw that there are enemies inside, so the fact that it's not being 
            -- captured can only be due to it being contested.
            --
            current_territory.relevant_cap_time.reset()
            for each player do
               if current_territory.shape_contains(current_player.biped) then 
                  script_widget[0].set_visibility(current_player, true)
                  if current_player.team == team[1] then 
                     current_flag.cap_timer_offense.set_rate(0%)
                  end
                  if current_player.team == team[0] then 
                     current_flag.cap_timer_defense.set_rate(0%)
                  end
                  if current_player.team == team[2] then 
                     current_flag.cap_timer_team_02.set_rate(0%)
                  end
                  if current_player.team == team[3] then 
                     current_flag.cap_timer_team_03.set_rate(0%)
                  end
               end
            end
         end
      end
   end
end

for each object with label "terr_object" do -- locked territory waypoint priority and timer management
   alias current_territory = global.object[0]
   alias current_flag      = global.object[1]
   --
   current_territory = current_object
   current_flag      = current_territory.decor_flag
   if current_territory.is_locked == 1 then 
      current_territory.set_waypoint_priority(normal)
      current_flag.cap_timer_offense.reset()
      current_flag.cap_timer_defense.reset()
      current_flag.cap_timer_team_03.reset()
      current_flag.cap_timer_team_02.reset()
      current_flag.timer[0].set_rate(0%) -- did they mean current_territory.timer[0]?
      current_territory.set_progress_bar(object.relevant_cap_time, no_one)
      current_territory.is_contested = 0
   end
end

for each object with label "terr_object" do -- handle territory capture
   alias current_territory = global.object[0]
   alias current_flag      = global.object[1]
   alias capturing_team    = global.team[1]
   --
   current_territory = current_object
   current_flag      = current_territory.decor_flag
   capturing_team    = current_territory.being_captured_by
   global.team[2]    = no_team -- may have been intended to be the team losing the territory, but isn't used now
   if  current_flag.timer[0].is_zero() -- timers are on the flags?
   or  current_flag.timer[1].is_zero()
   or  current_flag.timer[3].is_zero()
   or  current_flag.timer[2].is_zero()
   and not current_territory.being_captured_by == no_team
   then 
      current_territory.is_contested = 0
      send_incident(terr_captured, capturing_team, current_territory.team)
      game.show_message_to(capturing_team, none, "Territory %n captured!", current_territory.spawn_sequence)
      if global.team[2] != neutral_team then 
         game.show_message_to(current_territory.team, none, "Territory %n lost.", current_territory.spawn_sequence)
      end
      current_territory.team  = capturing_team
      current_territory.owner = capturing_team
      current_territory.relevant_cap_time.reset()
      current_flag.cap_timer_offense.reset()
      current_flag.cap_timer_team_02.reset()
      current_flag.cap_timer_defense.reset()
      current_flag.cap_timer_team_03.reset()
      if opt_symmetry == 1 then 
         current_territory.set_waypoint_priority(normal)
      end
      if opt_symmetry == 0 or opt_capture_lock == 1 then 
         current_territory.set_waypoint_priority(normal)
         current_territory.set_waypoint_icon(padlock)
         current_territory.set_shape_visibility(no_one)
         current_territory.set_progress_bar(object.relevant_cap_time, no_one)
         current_territory.is_contested = 0
         current_territory.is_locked    = 1
      end
      --
      -- Next up, we recolor the flag.
      --
      -- We already had the flag in a variable; this may be boilerplate i.e. Bungie's 
      -- language may allow (var.var.var = value) with the use of a temporary variable 
      -- automated by their compiler.
      --
      global.object[2] = current_territory.decor_flag -- recolor the flag
      global.object[2].team = capturing_team
      --
      for each player do
         if current_territory.shape_contains(current_player.biped) then 
            current_player.captures += 1
         end
      end
   end
end

do -- manage team scores
   scoring_interval.set_rate(-100%)
   for each team do
      if current_team.has_any_players() then 
         current_team.owned_terr_count     = 0
         current_team.contested_terr_count = 0
         for each object with label "terr_object" do
            if current_object.owner == current_team and current_object.is_locked == 1 or opt_capture_lock == 0 then 
               current_team.owned_terr_count += 1
               if current_object.is_contested == 1 then 
                  current_team.contested_terr_count += 1
               end
            end
         end
      end
   end
   for each team do
      if current_team.has_any_players() then 
         temp_int_00 = current_team.owned_terr_count
         if score_on_timer == 0 and opt_contested_scoring == 0 then 
            temp_int_00 -= current_team.contested_terr_count
         end
         if score_on_timer == 0 and scoring_interval.is_zero() then 
            current_team.score += temp_int_00
         end
         if score_on_timer == 1 then 
            current_team.score = temp_int_00
            if opt_symmetry == 0 and current_team == team[0] then 
               current_team.score = 0
            end
         end
      end
   end
   if scoring_interval.is_zero() then 
      scoring_interval.reset()
   end
end

for each object with label "terr_object" do -- allow Sudden Death if any territory is contested
   if sudden_death_enabled == 0 and current_object.is_contested == 1 then 
      sudden_death_enabled = 1
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() and sudden_death_enabled != 1 and game.grace_period_timer.is_zero() then 
   game.end_round()
end

for each team do -- asymmetric: end round if Offense takes all territories
   if opt_symmetry == 0 and current_team.has_any_players() and current_team == team[1] and current_team.score == opt_terr_count then 
      game.end_round()
   end
end

if opt_symmetry == 1 and opt_capture_lock == 1 then -- symmetric: end round if all territories are captured and locked
   alias unlocked_count = temp_int_00
   --
   unlocked_count = 0
   for each object with label "terr_object" do
      if current_object.is_locked == 0 then 
         unlocked_count += 1
      end
   end
   if unlocked_count == 0 then 
      game.end_round()
   end
end

do -- announce when a territory is contested
   alias current_territory = global.object[0]
   --
   terr_contest_announce_cooldown.set_rate(-100%)
   if terr_contest_announce_cooldown.is_zero() then 
      for each object with label "terr_object" do
         current_territory = current_object
         --
         if  current_territory.is_contested == 1
         and current_territory.announced_is_contested == 0
         and not current_territory.team == neutral_team
         or  current_territory.being_captured_by == no_team
         then 
            send_incident(terr_contested, all_players, all_players)
            current_territory.announced_is_contested = 1
            terr_contest_announce_cooldown.reset()
         end
      end
   end
end

for each team do -- symmetric team games: announce 30s to win and 60s to win
   alias threshold_30s = temp_int_00
   alias threshold_60s = temp_int_02
   if game.teams_enabled == 1 and opt_symmetry == 1 and current_team.has_any_players() then 
      threshold_30s = game.score_to_win
      threshold_60s = game.score_to_win
      threshold_30s -= 30
      threshold_60s -= 60
      if current_team.score >= threshold_60s and current_team.announced_60s_win == 0 and game.score_to_win > 60 then 
         send_incident(one_minute_team_win, current_team, all_players)
         current_team.announced_60s_win = 1
      end
      if current_team.score >= threshold_30s and current_team.announced_30s_win == 0 and game.score_to_win > 30 then 
         send_incident(half_minute_team_win, current_team, all_players)
         current_team.announced_30s_win = 1
      end
   end
end

-- Round timer management code below:
if not game.round_timer.is_zero() then 
   game.grace_period_timer = 0
end
if game.round_time_limit > 0 then 
   if not game.round_timer.is_zero() then 
      announced_sudden_death = 0
   end
   if game.round_timer.is_zero() then 
      if sudden_death_enabled == 1 then 
         game.sudden_death_timer.set_rate(-100%)
         game.grace_period_timer.reset()
         if announced_sudden_death == 0 then 
            send_incident(sudden_death, all_players, all_players)
            announced_sudden_death = 1
         end
         if game.sudden_death_time > 0 and game.grace_period_timer > game.sudden_death_timer then 
            game.grace_period_timer = game.sudden_death_timer
         end
      end
      if sudden_death_enabled == 0 then 
         game.grace_period_timer.set_rate(-100%)
         if game.grace_period_timer.is_zero() then 
            game.end_round()
         end
      end
      if game.sudden_death_timer.is_zero() then 
         game.end_round()
      end
   end
end
