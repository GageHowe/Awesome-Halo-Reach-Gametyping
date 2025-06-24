
enum style
   static -- the hill never moves
   moving -- the hill moves in order as determined by hill Spawn Sequence values
   random -- the hill moves in a random order
end

alias opt_hill_move_time = script_option[0]
alias opt_game_style     = script_option[1] -- see (style) enum
alias opt_multi_scoring  = script_option[2] -- Multiple Teammate Scoring
alias opt_super_shields  = script_option[3]

-- Unnamed Forge labels
alias all_jetpacks    = 0
alias all_armor_locks = 1
alias all_anti_zones  = 4
alias all_fire_vfx    = 5

alias hill_traits = script_traits[0]

alias time_in_hill = player.script_stat[0]

alias announced_game_start = player.number[4]
alias announce_start_timer = player.timer[3]

alias temp_int_00         = global.number[0]
alias current_hill_id     = global.number[1] -- used to activate hills in order
alias teams_in_hill       = global.number[2]
alias players_in_hill     = global.number[3]
alias temp_int_01         = global.number[4]
alias temp_int_02         = global.number[5] -- unused
alias after_first_hill    = global.number[6] -- used to avoid announcing "Hill Moved" for the first hill
alias hills_on_map        = global.number[7] -- if and only if we are not using the "Static" style
alias temp_int_03         = global.number[8]
alias temp_obj_00         = global.object[0]
alias current_hill        = global.object[1]
alias previous_hill       = global.object[2] -- do not allow the same hill to activate twice in a row in either random or ordered movement
alias temp_plr_00         = global.player[0]
alias hill_owner_to_announce = global.player[1]
alias temp_plr_01         = global.player[2]
alias award_next_point_to = global.player[3] -- team games: when multiple players are in the hill, take turns awarding a point to each of them. we alternate between players using player.award_team_points_timer
alias last_announced_hill_owner_player = global.player[4]
alias temp_tem_00         = global.team[0]
alias original_hill_owner = global.team[1] -- set at start of frame
alias last_announced_hill_owner_team = global.team[2] -- used to help announce when a new team takes control
alias temp_tem_01         = global.team[3]
alias loadout_cam_timer   = global.timer[0] -- used to delay the Hill Movement Timer until after the loadout camera
alias hill_move_timer     = global.timer[1]
alias hill_control_announce_cooldown = global.timer[2] -- don't announce "Hill controlled!" too often
alias sequence            = object.number[0] -- derived from Spawn Sequence, but normalized so that hills form a contiguous list starting at 1
alias sequence_validated  = object.number[1] -- used during the spawn sequence normalization process
alias ffa_owner           = object.player[0] -- FFA: player that controls the hill
alias hill_contest_announce_cooldown = object.timer[0] -- don't announce "Hill contested!" too often
alias minimum_control_timer = object.timer[1] -- used to start various functions only after an FFA hill has been controlled for at least one second
alias ui_move_timer         = object.timer[2] -- used to show the move timer in the waypoint
alias lifespan              = object.timer[3] -- for Super Shields VFX
alias ach_top_shot_count        = player.number[0]
alias ach_license_to_kill_count = player.number[1]
alias is_in_hill        = player.number[2]
alias announced_30s_win = player.number[6]
alias announced_60s_win = player.number[7]
alias hill_i_am_in      = player.object[0] -- set; never read
alias ach_paper_beats_rock_vuln_timer = player.timer[0]
alias hill_scoring_interval   = player.timer[1] -- every 1 second, increase Time In Hill stat by 1 and award 1 point
alias award_team_points_timer = player.timer[2] -- for team games, the amount of time the player has spent in the hill without receiving a point
alias anyone_in_hill    = team.number[0]
alias announced_30s_win = team.number[1]
alias announced_60s_win = team.number[2]
alias hill_scoring_interval = team.timer[0] -- every 1 second, increase Time In Hill stat by 1 and award 1 point

declare temp_int_00      with network priority local
declare current_hill_id  with network priority low -- hill sequence?
declare teams_in_hill    with network priority local
declare players_in_hill  with network priority local
declare temp_int_01      with network priority local
declare temp_int_02      with network priority local
declare after_first_hill with network priority local
declare hills_on_map     with network priority low
declare temp_int_03      with network priority local
declare temp_obj_00      with network priority local
declare current_hill     with network priority low
declare previous_hill    with network priority local
declare temp_plr_00      with network priority local
declare hill_owner_to_announce with network priority local
declare temp_plr_01            with network priority local
declare award_next_point_to    with network priority local
declare last_announced_hill_owner_player with network priority low
declare temp_tem_00            with network priority local
declare original_hill_owner    with network priority local -- NOT a temporary
declare last_announced_hill_owner_team with network priority low -- NOT a temporary
declare temp_tem_01            with network priority local
declare loadout_cam_timer = game.loadout_cam_time
declare hill_move_timer   = opt_hill_move_time
declare hill_control_announce_cooldown = 3
declare player.ach_top_shot_count        with network priority low
declare player.ach_license_to_kill_count with network priority low
declare player.is_in_hill with network priority local
declare player.number[3]         with network priority local -- unused
declare player.announced_game_start with network priority low
declare player.number[5]         with network priority low -- unused
declare player.announced_30s_win with network priority low
declare player.announced_60s_win with network priority low
declare player.hill_i_am_in      with network priority low
declare player.hill_scoring_interval = 1
declare player.announce_start_timer    = 5
declare object.sequence           with network priority low
declare object.sequence_validated with network priority low
declare object.ffa_owner          with network priority low
declare object.hill_contest_announce_cooldown = 3
declare object.minimum_control_timer          = 1
declare object.ui_move_timer                  = opt_hill_move_time
declare object.lifespan = 1
declare team.anyone_in_hill    with network priority local
declare team.announced_30s_win with network priority low
declare team.announced_60s_win with network priority low
declare team.hill_scoring_interval = 1

for each player do -- award Dive Bomber achievement as appropriate
   alias killer    = temp_plr_00
   alias killer_aa = temp_obj_00
   alias death_mod = temp_int_00
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
   alias killer    = temp_plr_00
   alias death_mod = temp_int_00
   if current_player.killer_type_is(kill) then 
      death_mod = 0
      death_mod = current_player.try_get_death_damage_mod()
      if death_mod == enums.damage_reporting_modifier.sticky then 
         killer = no_player
         killer = current_player.try_get_killer()
         if killer.killer_type_is(suicide) then 
            send_incident(dlc_achieve_2, current_player, current_player, 68)
         end
      end
   end
end

for each player do -- manage and award Top Shot achievement as appropriate
   alias killer    = temp_plr_00
   alias death_mod = temp_int_00
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.ach_top_shot_count = 0
      if current_player.killer_type_is(kill) then 
         killer    = no_player
         killer    = current_player.try_get_killer()
         death_mod = 0
         death_mod = current_player.try_get_death_damage_mod()
         if death_mod != enums.damage_reporting_modifier.headshot then 
            killer.ach_top_shot_count = 0
         end
         if death_mod == enums.damage_reporting_modifier.headshot then 
            killer.ach_top_shot_count += 1
            if killer.ach_top_shot_count > 2 then 
               send_incident(dlc_achieve_2, killer, killer, 62)
            end
         end
      end
   end
end

for each player do -- manage and award License To Kill achievement as appropriate
   alias killer    = temp_plr_00
   alias death_mod = temp_int_00
   alias vehicle   = temp_obj_00
   if current_player.killer_type_is(kill) then 
      killer    = no_player
      killer    = current_player.try_get_killer()
      death_mod = 0
      death_mod = current_player.try_get_death_damage_mod()
      vehicle   = no_object
      vehicle   = killer.try_get_vehicle()
      if vehicle != no_object and death_mod == enums.damage_reporting_modifier.splatter then 
         killer.ach_license_to_kill_count += 1
         if killer.ach_license_to_kill_count > 4 then 
            send_incident(dlc_achieve_2, killer, killer, 66)
         end
      end
   end
end

for each player do -- manage timing for the Paper Beats Rock achievement
   alias current_aa = temp_obj_00
   --
   current_aa = no_object
   current_aa = current_player.try_get_armor_ability()
   if current_aa.has_forge_label(all_armor_locks) and current_aa.is_in_use() then 
      current_player.ach_paper_beats_rock_vuln_timer = 4
      current_player.ach_paper_beats_rock_vuln_timer.set_rate(-100%)
   end
end
for each player do -- award Paper Beats Rock achievement as appropriate
   alias killer    = temp_plr_00
   alias death_mod = temp_int_00
   if current_player.killer_type_is(kill) and not current_player.ach_paper_beats_rock_vuln_timer.is_zero() then 
      death_mod = 0
      death_mod = current_player.try_get_death_damage_mod()
      if death_mod == enums.damage_reporting_modifier.assassination then 
         killer = no_player
         killer = current_player.try_get_killer()
         send_incident(dlc_achieve_2, killer, killer, 60)
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

on init: do
   after_first_hill = 0
   loadout_cam_timer.reset()
end

-- The next two triggers normalize the hill sequence values.
if opt_game_style != style.static then 
   hills_on_map = 0
   for each object with label "koth_hill" do
      hills_on_map += 1
      current_object.sequence_validated = 0
      if current_object.sequence == 0 then -- if this hill hasn't had its order normalized yet
         current_object.sequence = current_object.spawn_sequence
      end
   end
end
if opt_game_style != style.static then
   --
   -- Map variant authors can use a hill's Spawn Sequence to determine 
   -- the order in which the hill activates during Crazy King. However, 
   -- they may leave gaps in their numbering, or they may start their 
   -- numbering at a value other than 1. To keep things relatively 
   -- simple during hill selection, let's renumber all hills on the 
   -- map to normalize them to a contiguous range of numbers starting 
   -- at 1.
   --
   -- If there are any gaps in the list, we'll fill those with any 
   -- hills we renumber first; otherwise, we'll add renumbered hills to 
   -- the end of the list.
   --
   alias current_sequence  = temp_int_00
   alias value_is_in_use   = temp_int_03 -- there exists no hill with spawn sequence (current_sequence)
   alias reorder_this_hill = temp_obj_00
   --
   current_sequence  = 1
   value_is_in_use   = 0
   reorder_this_hill = no_object
   for each object with label "koth_hill" do -- these loops automatically iterate in Spawn Sequence order
      --
      -- We don't actually care about the object that the outer loop loops over; rather, we're 
      -- just looping the value (current_sequence) over the numeric range from 1 to the number 
      -- of objects with the label.
      --
      value_is_in_use   = 0
      reorder_this_hill = no_object
      for each object with label "koth_hill" do
         if value_is_in_use == 0 and current_object.sequence_validated == 0 then 
            if current_object.sequence == current_sequence then 
               value_is_in_use = 1
               current_object.sequence_validated = 1 -- ensure we don't re-process this hill
            end
            if  current_object.sequence > hills_on_map
            or  current_object.sequence < current_sequence
            and reorder_this_hill == no_object -- we can only reorder one hill at a time
            then 
               reorder_this_hill = current_object
            end
         end
      end
      if value_is_in_use == 0 and reorder_this_hill != no_object then
         --
         -- We found a hill with an invalid value, and the value we're currently using isn't 
         -- already in use by another hill.
         --
         reorder_this_hill.sequence = current_sequence
         reorder_this_hill.sequence_validated = 1
      end
      current_sequence += 1
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

for each player do -- round card and announce game start
   current_player.announce_start_timer.set_rate(-100%)
   loadout_cam_timer.set_rate(-100%)
   if game.score_to_win != 0 and game.teams_enabled == 1 then 
      current_player.set_round_card_title("Control the hill for your team.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win != 0 and game.teams_enabled == 0 then 
      current_player.set_round_card_title("Control the hill to earn points.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win == 0 and game.teams_enabled == 1 then 
      current_player.set_round_card_title("Control the hill for your team.")
   end
   if game.score_to_win == 0 and game.teams_enabled == 0 then 
      current_player.set_round_card_title("Control the hill to earn points.")
   end
   if current_player.announce_start_timer.is_zero() and current_player.announced_game_start == 0 then 
      send_incident(koth_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

on host migration: do
   for each object with label all_anti_zones do
      current_object.delete()
   end
   award_next_point_to = no_player
   for each player do
      current_player.award_team_points_timer = 0
      current_player.hill_scoring_interval.reset()
      current_player.hill_scoring_interval.set_rate(0%)
   end
   for each team do
      current_team.hill_scoring_interval.reset()
      current_team.hill_scoring_interval.set_rate(0%)
   end
end

if opt_game_style != style.static then -- handle hill movement
   --
   -- If we ever need to move the hill early, then we'll just force the 
   -- timer to zero when we detect whatever circumstance we need to react 
   -- to.
   --
   if loadout_cam_timer.is_zero() then 
      hill_move_timer.set_rate(-100%)
   end
   if hill_move_timer.is_zero() then
      alias hills_on_map = temp_int_01
      --
      hill_move_timer = opt_hill_move_time
      previous_hill   = current_hill
      current_hill    = no_object
      hills_on_map = 0
      for each object with label "koth_hill" do
         hills_on_map += 1
      end
      if opt_game_style == style.random then 
         current_hill = get_random_object("koth_hill", previous_hill)
         if after_first_hill == 0 and current_hill.sequence != 1 then
            --
            -- For the initial hill, prefer a hill with a spawn sequence of 1; if 
            -- no such hill exists, select the hill with the lowest spawn sequence.
            --
            current_hill = no_object
         end
         if current_hill != no_object then 
            if after_first_hill == 1 then 
               send_incident(hill_moved, all_players, all_players)
            end
            after_first_hill = 1
         end
      end
      if opt_game_style == style.moving then
         --
         -- Advance (current_hill_id) and wrap it if needed. We cleared (current_hill) 
         -- earlier; the next trigger will try to select a new hill with the new ID.
         --
         current_hill_id += 1
         if current_hill_id > hills_on_map then 
            current_hill_id = 0
         end
      end
   end
end
if current_hill == no_object then -- sequential hill move
   --
   -- The (current_hill) variable will be zero only in two cases: we are using a 
   -- random hill, but we failed to pick a valid one; or we are using an ordered 
   -- hill. See previous block.
   --
   temp_int_01 = -1 -- not used here
   for each object with label all_anti_zones do
      current_object.delete()
   end
   for each object with label "koth_hill" do
      if current_hill == no_object then 
         temp_int_01 += 1
         if current_object.sequence == current_hill_id and not current_object == previous_hill then 
            current_hill      = current_object
            current_hill.team = neutral_team
            hill_owner_to_announce = no_player
            if after_first_hill == 1 then 
               send_incident(hill_moved, all_players, all_players)
               last_announced_hill_owner_player = no_player
               last_announced_hill_owner_team = no_team
            end
            temp_obj_00 = no_object
            temp_obj_00 = current_object.place_at_me(respawn_zone_weak_anti, none, never_garbage_collect, 0, 0, 0, none)
            temp_obj_00.team = neutral_team
            previous_hill = current_object
            after_first_hill = 1
         end
      end
   end
   if current_hill == no_object then
      --
      -- Failed to select a hill. Force the timer to zero so that we try again 
      -- next frame. (As a bonus, that'll also increment the current hill number, 
      -- so if there are any gaps in the list due to hills being destroyed, then 
      -- we'll just skip over them.)
      --
      hill_move_timer = 0
   end
end

for each object with label "koth_hill" do
   current_object.set_shape_visibility(no_one)
   current_object.set_waypoint_visibility(no_one)
   current_object.ui_move_timer = hill_move_timer
   current_object.minimum_control_timer.set_rate(-100%)
   if game.teams_enabled == 0 then
      current_object.apply_shape_color_from_player_member(object.ffa_owner)
   end
end

on object death: do -- handle hill death
   if killed_object == current_hill then -- if the hill is destroyed, deactivate it
      current_hill = no_object
      if opt_game_style != style.static then 
         hill_move_timer = 0 -- ...and force instant hill movement
      end
   end
   for each object with label "koth_hill" do -- if the hill is destroyed, delete it outright
      if current_object == killed_object then
         current_object.delete()
      end
   end
end

do
   teams_in_hill       = 0
   players_in_hill     = 0
   original_hill_owner = current_hill.team
   current_hill.team   = neutral_team
   current_hill.ffa_owner = no_player
   --
   current_hill.set_waypoint_visibility(everyone)
   current_hill.set_waypoint_icon(crown)
   if opt_game_style != style.static then 
      current_hill.set_waypoint_timer(object.ui_move_timer)
   end
   current_hill.set_shape_visibility(everyone)
   current_hill.set_waypoint_priority(high)
   if hill_move_timer < 5 then -- blink the hill waypoint when it's about to move
      current_hill.set_waypoint_priority(blink)
   end
end

-- Apply Hill Traits, and for team games, count the number of teams in the hill.
for each team do
   current_team.anyone_in_hill = 0
end
for each player do
   alias player_team = temp_tem_00
   --
   current_player.is_in_hill = 0
   if current_hill.shape_contains(current_player.biped) then 
      player_team = current_player.team
      current_player.apply_traits(hill_traits)
      players_in_hill += 1
      current_player.is_in_hill = 1
      if game.teams_enabled == 1 and not player_team.anyone_in_hill == 1 then 
         player_team.anyone_in_hill = 1
         teams_in_hill += 1
      end
   end
end
for each team do -- manage team.hill_scoring_interval rate and current_hill.team
   if game.teams_enabled == 1 then 
      current_team.hill_scoring_interval.set_rate(0%)
      if current_team.anyone_in_hill == 1 and teams_in_hill == 1 then 
         current_team.hill_scoring_interval.set_rate(-100%)
         current_hill.team = current_team
      end
   end
end

for each player do
   current_player.hill_scoring_interval.set_rate(0%)
   current_player.hill_i_am_in = no_object
   if current_player.is_in_hill == 0 then 
      current_player.award_team_points_timer.reset()
   end
   if current_player.is_in_hill == 1 then
      alias factions_in_hill = temp_int_00
      --
      factions_in_hill = teams_in_hill
      if game.teams_enabled == 0 then 
         factions_in_hill = players_in_hill
      end
      if not factions_in_hill == 1 then -- hill is contested
         current_hill.minimum_control_timer.reset()
      end
      if factions_in_hill == 1 then -- hill is controlled
         current_player.hill_i_am_in = current_hill
         current_hill.ffa_owner = current_player
         current_hill.minimum_control_timer.set_rate(-100%)
         if current_hill.minimum_control_timer.is_zero() then 
            hill_owner_to_announce = current_player
            current_player.hill_scoring_interval.set_rate(-100%)
            temp_tem_01 = current_player.team
            temp_tem_01.hill_scoring_interval.set_rate(-100%)
            current_hill.team = current_player.team
         end
      end
   end
end
for each player do -- FFA: manage scoring and Time In Hill stat
   if game.teams_enabled == 0 and current_player.hill_scoring_interval.is_zero() then 
      current_player.score += 1
      current_player.time_in_hill += 1
      current_player.hill_scoring_interval.reset()
   end
end

-- Team: manage scoring and Time In Hill stat
for each player do
   current_player.award_team_points_timer.set_rate(0%)
   if current_hill.shape_contains(current_player.biped) then 
      current_player.award_team_points_timer.set_rate(100%)
   end
end
for each team do -- if this team has control of the hill, take turns awarding a point to each team member in the hill
   alias relevant_team = temp_tem_01
   if game.teams_enabled == 1 and current_team.has_any_players() then 
      relevant_team = current_team
      --
      -- Identify the player that has spent the longest amount of time in the hill 
      -- without receiving a point. We'll give them a point next. (Of course, if 
      -- Multiple Teammate Scoring is enabled, then we'll give everyone else a 
      -- point, too.)
      --
      for each player do
         if current_player.team == relevant_team and current_hill.shape_contains(current_player.biped) then 
            players_in_hill += 1
            if current_player.award_team_points_timer > award_next_point_to.award_team_points_timer then 
               award_next_point_to = current_player
            end
         end
      end
      if current_team.hill_scoring_interval.is_zero() then 
         current_team.hill_scoring_interval.reset()
         --
         -- Manage the Time In Hill stat, and award points as appropriate.
         --
         for each player do
            if current_player.team == relevant_team and current_hill.shape_contains(current_player.biped) then 
               current_player.time_in_hill += 1
               current_player.hill_scoring_interval = 1
               if current_player == award_next_point_to or opt_multi_scoring == 1 then 
                  current_player.score += 1
                  current_player.award_team_points_timer.reset()
               end
            end
         end
      end
   end
end

if  game.teams_enabled  == 0
and players_in_hill     == 1 -- hill is controlled
and original_hill_owner == neutral_team -- the hill wasn't controlled at the start of this frame
and current_hill.minimum_control_timer.is_zero()
then 
   current_hill.hill_contest_announce_cooldown.set_rate(-100%)
end

if  game.teams_enabled  == 1
and teams_in_hill       == 1 -- hill is controlled
and original_hill_owner == neutral_team -- the hill wasn't controlled at the start of this frame
and current_hill.minimum_control_timer.is_zero()
then 
   temp_tem_01 = award_next_point_to.team
   current_hill.hill_contest_announce_cooldown.set_rate(-100%)
end

if  game.teams_enabled == 0
and players_in_hill > 1 -- hill is contested
and not original_hill_owner == neutral_team -- the hill wasn't contested or unoccupied at the start of this frame
and current_hill.hill_contest_announce_cooldown.is_zero() -- don't announce too frequently
then 
   for each player do
      if current_hill.shape_contains(current_player.biped) then 
         send_incident(hill_contested, current_player, no_player)
      end
   end
   current_hill.hill_contest_announce_cooldown.reset()
   current_hill.hill_contest_announce_cooldown.set_rate(-100%)
   current_hill.player[0] = no_player
end

if  game.teams_enabled == 1
and teams_in_hill > 1 -- hill is contested
and not original_hill_owner == neutral_team -- the hill wasn't contested or unoccupied at the start of this frame
and current_hill.hill_contest_announce_cooldown.is_zero() then -- don't announce too frequently
   for each player do
      if current_hill.shape_contains(current_player.biped) then 
         send_incident(hill_contested_team, current_player, no_player)
      end
   end
   current_hill.hill_contest_announce_cooldown.reset()
   current_hill.hill_contest_announce_cooldown.set_rate(-100%)
   current_hill.player[0] = no_player
end

do -- announce when someone takes control of the hill
   hill_control_announce_cooldown.set_rate(-100%)
   if hill_control_announce_cooldown.is_zero() then 
      if game.teams_enabled == 0 then 
         if players_in_hill <= 0 then 
            last_announced_hill_owner_player = no_player
         end
         if players_in_hill == 1 and hill_owner_to_announce != last_announced_hill_owner_player then 
            send_incident(hill_controlled, hill_owner_to_announce, all_players)
            last_announced_hill_owner_player = hill_owner_to_announce
            hill_control_announce_cooldown.reset()
         end
      end
      if game.teams_enabled == 1 then
         alias hill_is_contested     = temp_int_00
         alias hill_team_to_announce = temp_tem_01
         --
         hill_team_to_announce = no_team
         hill_is_contested     = 0
         for each player do
            if current_hill.shape_contains(current_player.biped) then 
               hill_is_contested = 1
               if hill_team_to_announce == current_player.team or hill_team_to_announce == no_team then 
                  hill_team_to_announce = current_player.team
                  hill_is_contested = 0
               end
            end
         end
         if teams_in_hill <= 0 then 
            last_announced_hill_owner_team = no_team
         end
         if hill_is_contested != 1 and hill_team_to_announce != last_announced_hill_owner_team then 
            send_incident(hill_controlled_team, hill_team_to_announce, all_players)
            last_announced_hill_owner_team = hill_team_to_announce
            hill_control_announce_cooldown.reset()
         end
      end
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each player do -- medal for killing an enemy while inside the hill
   alias killer = temp_plr_01
   if current_player.killer_type_is(kill) then 
      killer = current_player.try_get_killer()
      if killer.is_in_hill == 1 then 
         send_incident(kill_within_hill, killer, current_player)
      end
   end
end

do -- announce at 60 seconds from victory and 30 seconds from victory
   alias threshold_30s = temp_int_00
   alias threshold_60s = temp_int_03
   for each player do
      if game.teams_enabled == 0 then 
         threshold_30s = game.score_to_win
         threshold_60s = game.score_to_win
         threshold_30s -= 30
         threshold_60s -= 60
         if current_player.score >= threshold_60s and game.score_to_win > 60 and current_player.announced_60s_win == 0 then 
            send_incident(one_minute_win, current_player, all_players)
            current_player.announced_60s_win = 1
         end
         if current_player.score >= threshold_30s and game.score_to_win > 30 and current_player.announced_30s_win == 0 then 
            send_incident(half_minute_win, current_player, all_players)
            current_player.announced_30s_win = 1
         end
      end
   end
   for each team do
      if game.teams_enabled == 1 and current_team.has_any_players() then 
         threshold_30s = game.score_to_win
         threshold_60s = game.score_to_win
         threshold_30s -= 30
         threshold_60s -= 60
         if current_team.score >= threshold_60s and game.score_to_win > 60 and current_team.announced_60s_win == 0 then 
            send_incident(one_minute_team_win, current_team, all_players)
            current_team.announced_60s_win = 1
         end
         if current_team.score >= threshold_30s and game.score_to_win > 30 and current_team.announced_30s_win == 0 then 
            send_incident(half_minute_team_win, current_team, all_players)
            current_team.announced_30s_win = 1
         end
      end
   end
end

-- Super Shields:
for each player do
   alias current_shields = temp_int_00
   alias vfx_probability = temp_int_03
   if opt_super_shields == 1 then 
      current_shields = 0
      current_shields = current_player.biped.shields
      if current_shields > 100 then 
         vfx_probability = 0
         vfx_probability = rand(10)
         if vfx_probability <= 2 then -- 30% chance to spawn a particle emitter
            current_player.biped.place_at_me(particle_emitter_fire, none, never_garbage_collect, 0, 0, 0, none)
         end
      end
   end
end
for each object with label all_fire_vfx do
   current_object.lifespan.set_rate(-100%)
   if current_object.lifespan.is_zero() then 
      current_object.delete()
   end
end
