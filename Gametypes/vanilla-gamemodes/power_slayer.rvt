
enum hidden_gametype
   normal
   classic
   pro
   covy
   swat = 7
   bro
   power
end

alias opt_hidden_gametype = script_option[0]
alias opt_kill_points     = script_option[1]
alias opt_death_points    = script_option[2]
alias opt_suicide_points  = script_option[3]
alias opt_betrayal_points = script_option[4]
alias opt_headshot_bonus  = script_option[5]
alias opt_pummel_bonus    = script_option[6]
alias opt_assassin_bonus  = script_option[7]
alias opt_splatter_bonus  = script_option[8]
alias opt_sticky_bonus    = script_option[9]
alias opt_spree_bonus     = script_option[10]
alias opt_powerup_time    = script_option[11]

-- Unnamed Forge labels:
alias all_health_packs = 1
alias all_fire_vfx     = 3

alias kill_traits     = script_traits[0]
alias pummel_traits   = script_traits[1]
alias assassin_traits = script_traits[2]
alias headshot_traits = script_traits[3]

alias ui_powerup_time = script_widget[0]

alias rating_stat = player.script_stat[0]

alias announced_game_start = player.number[0]
alias announce_start_timer = player.timer[0]

alias temp_int_00  = global.number[0]
alias temp_int_01  = global.number[1]
alias temp_int_02  = global.number[2]
alias temp_int_03  = global.number[3]
alias temp_int_04  = global.number[4] -- used by accident; the sole reference seems meant to have been temp_int_03 instead
alias temp_obj_00  = global.object[0]
alias temp_plr_00  = global.player[0]
alias vfx_lifespan = object.timer[0]
alias has_powerup_pummel   = player.number[1]
alias has_powerup_assassin = player.number[2]
alias has_powerup_headshot = player.number[3]
alias has_powerup_kill     = player.number[4]
alias last_biped           = player.object[0]
alias powerup_time         = player.timer[1] -- remaining time for the player's powerups

declare temp_int_00 with network priority local
declare temp_int_01 with network priority local
declare temp_int_02 with network priority local
declare temp_int_03 with network priority local
declare temp_int_04 with network priority local
declare temp_obj_00 with network priority local
declare temp_plr_00 with network priority local
declare player.announced_game_start with network priority low
declare player.has_powerup_pummel   with network priority low
declare player.has_powerup_assassin with network priority low
declare player.has_powerup_headshot with network priority low
declare player.has_powerup_kill     with network priority low
declare player.last_biped with network priority low
declare player.announce_start_timer = 5
declare object.vfx_lifespan = 1

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
end

for each player do -- loadout palettes
   if current_player.is_elite() then 
      current_player.set_loadout_palette(elite_tier_1)
   end
   if not current_player.is_elite() then 
      current_player.set_loadout_palette(spartan_tier_1)
   end
end

for each player do -- UI setup and round card
   ui_powerup_time.set_text("Powerup fading in %s", hud_player.powerup_time)
   ui_powerup_time.set_visibility(current_player, false)
   if game.score_to_win != 0 and game.teams_enabled == 1 then 
      current_player.set_round_card_title("Kill players on the enemy team.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win != 0 and game.teams_enabled == 0 then 
      current_player.set_round_card_title("Score points by killing other players.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win == 0 and game.teams_enabled == 1 then 
      current_player.set_round_card_title("Kill players on the enemy team.")
   end
   if game.score_to_win == 0 and game.teams_enabled == 0 then 
      current_player.set_round_card_title("Score points by killing other players.")
   end
end

for each team do -- Buddy Slayer spawning mode setup
   if opt_hidden_gametype == hidden_gametype.bro then 
      current_team.set_co_op_spawning(true)
   end
end

for each object with label "bro_spawn_loc" do -- Buddy Slayer spawn permission setup
   if opt_hidden_gametype == hidden_gametype.bro then 
      current_object.set_invincibility(1)
      current_object.set_pickup_permissions(no_one)
      current_object.set_spawn_location_fireteams(all)
      current_object.set_spawn_location_permissions(allies)
      for each player do
         if current_object.team == current_player.team then 
            temp_int_01 = 0
            current_player.set_primary_respawn_object(current_object)
         end
      end
   end
end

for each player do -- announce game start
   current_player.announce_start_timer.set_rate(-100%)
   current_player.rating_stat = current_player.rating -- show Arena rating in the post-game carnage report
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      if opt_hidden_gametype == hidden_gametype.swat then 
         send_incident(swat_game_start, current_player, no_player)
      end
      if not opt_hidden_gametype == hidden_gametype.swat then 
         send_incident(game_start_slayer, current_player, no_player)
      end
      current_player.announced_game_start = 1
   end
end

-- Delete all Health Packs in Covy Slayer and SWAT:
if opt_hidden_gametype == hidden_gametype.covy or opt_hidden_gametype == hidden_gametype.swat then 
   for each object with label all_health_packs do
      current_object.delete()
   end
end

for each player do -- track player's last biped for achievement stuff; most kill/death processing
   alias killer      = temp_plr_00
   alias damage_mod  = temp_int_00
   alias damage_type = temp_int_02
   --
   temp_int_01 = 0
   damage_type = 0
   --
   -- If one player kills another player with a DMR, we want to check how far apart 
   -- they were so we can award the Cross-Mappin' achievement if appropriate. However, 
   -- a player's biped property is cleared when they die. The biped itself is still 
   -- accessible to script, but not via the property. Accordingly, we want to remember 
   -- a player's last known biped so we can make use of it when the player dies:
   --
   temp_obj_00 = current_player.biped
   if temp_obj_00 != no_object then 
      current_player.last_biped = temp_obj_00
   end
   --
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then
      current_player.score += opt_death_points
      --
      -- Dying revokes all of your powerups:
      --
      current_player.has_powerup_kill     = 0
      current_player.has_powerup_pummel   = 0
      current_player.has_powerup_headshot = 0
      current_player.has_powerup_assassin = 0
      current_player.powerup_time = 0
      --
      if current_player.killer_type_is(kill) then 
         killer = current_player.try_get_killer()
         killer.score += opt_kill_points
         killer.has_powerup_kill = 1
         do -- handle DLC achievement: Cross-Mappin'
            alias distance = temp_int_03
            --
            distance    = 0
            temp_int_04 = 0 -- I think they meant to clear (damage_type) here
            damage_type = current_player.try_get_death_damage_type()
            if damage_type == enums.damage_reporting_type.dmr then 
               distance = current_player.last_biped.get_distance_to(killer.last_biped)
               if distance > 400 then 
                  send_incident(dlc_achieve_5, killer, current_player)
               end
            end
         end
         damage_mod  = current_player.try_get_death_damage_mod()
         temp_int_01 = killer.get_spree_count()
         damage_type = current_player.try_get_death_damage_type()
         do
            temp_int_01 %= 5
            if temp_int_01 == 0 then 
               killer.score += opt_spree_bonus
            end
         end
         if damage_mod == enums.damage_reporting_modifier.pummel then 
            killer.score += opt_pummel_bonus
            killer.has_powerup_pummel = 1
         end
         if damage_mod == enums.damage_reporting_modifier.assassination then 
            killer.score += opt_assassin_bonus
            killer.has_powerup_assassin = 1
         end
         if damage_mod == enums.damage_reporting_modifier.splatter then 
            killer.score += opt_splatter_bonus
         end
         if damage_mod == enums.damage_reporting_modifier.sticky then 
            killer.score += opt_sticky_bonus
         end
         if damage_mod == enums.damage_reporting_modifier.headshot then 
            killer.score += opt_headshot_bonus
            killer.has_powerup_headshot = 1
         end
         if damage_type == enums.damage_reporting_type.frag_grenade then 
            killer.frag_grenades += 1
            game.show_message_to(killer, boneyard_generator_power_down, "You have received additional grenades!")
         end
         if damage_type == enums.damage_reporting_type.plasma_grenade then 
            killer.plasma_grenades += 1
            game.show_message_to(killer, boneyard_generator_power_down, "You have received additional grenades!")
         end
         if killer.has_powerup_pummel   == 1
         or killer.has_powerup_assassin == 1
         or killer.has_powerup_headshot == 1
         or killer.has_powerup_kill     == 1
         then 
            killer.powerup_time = opt_powerup_time
         end
      end
   end
end

for each player do -- award suicide points
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) and not current_player.killer_type_is(kill) and not current_player.killer_type_is(betrayal) then 
      current_player.score += opt_suicide_points
   end
end

for each player do -- award betrayal points
   alias killer = temp_plr_00
   if current_player.killer_type_is(betrayal) then 
      killer = current_player.try_get_killer()
      killer.score += opt_betrayal_points
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each player do
   current_player.powerup_time.set_rate(-100%)
   if current_player.powerup_time.is_zero() then -- active powerups expiring
      current_player.has_powerup_kill     = 0
      current_player.has_powerup_pummel   = 0
      current_player.has_powerup_assassin = 0
      current_player.has_powerup_headshot = 0
   end
   if not current_player.powerup_time.is_zero() then -- apply powerup traits
      alias allow_vfx = temp_int_01
      alias vfx_odds  = temp_int_02
      --
      ui_powerup_time.set_visibility(current_player, true)
      allow_vfx = 0
      if current_player.has_powerup_kill == 1 then 
         current_player.apply_traits(kill_traits)
         if 1 == 1 then 
            allow_vfx = 1
         end
      end
      if current_player.has_powerup_pummel == 1 then 
         current_player.apply_traits(pummel_traits)
         if 1 == 1 then 
            allow_vfx = 1
         end
      end
      if current_player.has_powerup_assassin == 1 then 
         current_player.apply_traits(assassin_traits)
         if 1 == 1 then 
            allow_vfx = 1
         end
      end
      if current_player.has_powerup_headshot == 1 then 
         current_player.apply_traits(headshot_traits)
         if 1 == 1 then 
            allow_vfx = 1
         end
      end
      if allow_vfx == 1 then 
         vfx_odds = 0
         vfx_odds = rand(10)
         if vfx_odds <= 2 then -- 30% chance
            current_player.biped.place_at_me(particle_emitter_fire, none, never_garbage_collect, 0, 0, 0, none)
         end
      end
   end
end
for each object with label all_fire_vfx do -- clean up VFX
   current_object.vfx_lifespan.set_rate(-100%)
   if current_object.vfx_lifespan.is_zero() then 
      current_object.delete()
   end
end
