# simplesit-2.0 update .class

package net.apcat.simplesit;

import java.io.File;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.UUID;
import java.util.logging.Level;
import java.util.logging.Logger;
import net.apcat.simplesit.commands.LayCommand;
import net.apcat.simplesit.commands.SitCommand;
import net.apcat.simplesit.listeners.PlayerArmorStandManipulate;
import net.apcat.simplesit.listeners.PlayerDeath;
import net.apcat.simplesit.listeners.PlayerJoin;
import net.apcat.simplesit.listeners.PlayerQuit;
import net.apcat.simplesit.listeners.PlayerTeleport;
import net.apcat.simplesit.tasks.CheckForUpdatesTask;
import net.apcat.simplesit.tasks.LayTask;
import net.apcat.simplesit.tasks.RotateSeatTask;
import net.apcat.simplesit.utils.Text;
import net.apcat.simplesit.utils.Utils;
import org.bukkit.Bukkit;
import org.bukkit.command.PluginCommand;
import org.bukkit.configuration.file.FileConfiguration;
import org.bukkit.entity.ArmorStand;
import org.bukkit.permissions.Permission;
import org.bukkit.permissions.PermissionDefault;
import org.bukkit.plugin.PluginManager;
import org.bukkit.plugin.java.JavaPlugin;

public class SimpleSit
  extends JavaPlugin
{
  private Map<UUID, ArmorStand> seats = new HashMap();
  private Map<UUID, LayTask> laying = new HashMap();
  private boolean checkForUpdates;
  private Permission sitPermission;
  private Permission layPermission;
  private String sitDownMessage;
  private String sitUpMessage;
  private String sitFailMessage;
  private String layDownMessage;
  private String layUpMessage;
  private String layFailMessage;
  private String layOverlapMessage;
  
  public void onEnable()
  {
    registerConfig();
    registerPermissions();
    registerCommands();
    registerListeners();
    new RotateSeatTask(this);
    if (this.checkForUpdates) {
      new CheckForUpdatesTask(Bukkit.getConsoleSender());
    }
  }
  
  public void onDisable()
  {
    Object[] arrayOfObject;
    int j = (arrayOfObject = this.seats.keySet().toArray()).length;
    for (int i = 0; i < j; i++)
    {
      Object uuid = arrayOfObject[i];
      SimpleSitPlayer player = new SimpleSitPlayer(Bukkit.getPlayer((UUID)uuid));
      player.setSitting(false);
    }
    j = (arrayOfObject = this.laying.keySet().toArray()).length;
    for (i = 0; i < j; i++)
    {
      Object uuid = arrayOfObject[i];
      SimpleSitPlayer player = new SimpleSitPlayer(Bukkit.getPlayer((UUID)uuid));
      player.setLaying(false);
    }
  }
  
  public void convertConfig()
  {
    boolean convert = false;
    String oldPermissionDefault = null;String oldSitDownMessage = null;String oldSitUpMessage = null;String oldSitFailMessage = null;
    if (getConfig().contains("checkForUpdates"))
    {
      convert = true;
      if (getConfig().contains("sitPermissionDefault")) {
        oldPermissionDefault = getConfig().getString("sitPermissionDefault");
      }
      if (getConfig().contains("sitDown")) {
        oldSitDownMessage = getConfig().getString("sitDown");
      }
      if (getConfig().contains("sitFail")) {
        oldSitFailMessage = getConfig().getString("sitFail");
      }
      if (getConfig().contains("sitUp")) {
        oldSitUpMessage = getConfig().getString("sitUp");
      }
      File configFile = new File(getDataFolder(), "config.yml");
      configFile.delete();
    }
    saveDefaultConfig();
    reloadConfig();
    if (!convert) {
      return;
    }
    if (oldPermissionDefault != null) {
      getConfig().set("sit-permission-default", oldPermissionDefault);
    }
    if (oldSitDownMessage != null) {
      getConfig().set("sitdown-message", oldSitDownMessage);
    }
    if (oldSitFailMessage != null) {
      getConfig().set("sitfail-message", oldSitFailMessage);
    }
    if (oldSitUpMessage != null) {
      getConfig().set("situp-message", oldSitUpMessage);
    }
    saveConfig();
    reloadConfig();
  }
  
  private void registerConfig()
  {
    convertConfig();
    this.checkForUpdates = getConfig().getBoolean("check-for-updates");
    
    this.sitDownMessage = Utils.color(getConfig().getString("sitdown-message"));
    this.sitUpMessage = Utils.color(getConfig().getString("situp-message"));
    this.sitFailMessage = Utils.color(getConfig().getString("sitfail-message"));
    
    this.layDownMessage = Utils.color(getConfig().getString("laydown-message"));
    this.layUpMessage = Utils.color(getConfig().getString("layup-message"));
    this.layFailMessage = Utils.color(getConfig().getString("layfail-message"));
    this.layOverlapMessage = Utils.color(getConfig().getString("layoverlap-message"));
  }
  
  private void registerPermissions()
  {
    this.sitPermission = getPermission("simplesit.sit", "sit-permission-default");
    this.layPermission = getPermission("simplesit.lay", "lay-permission-default");
  }
  
  private Permission getPermission(String permissionString, String location)
  {
    Permission permission = new Permission(permissionString);
    String permissionDefault = getConfig().getString(location);
    if (!Utils.isPermissionDefault(permissionDefault))
    {
      sendConfigError(Text.INVALID_PERMISSION_DEFAULT.format(new Object[] { location, permissionDefault }), Level.WARNING);
      permission.setDefault(PermissionDefault.TRUE);
    }
    else
    {
      permission.setDefault(PermissionDefault.valueOf(permissionDefault.toUpperCase()));
    }
    Bukkit.getPluginManager().addPermission(permission);
    return permission;
  }
  
  private void registerCommands()
  {
    getCommand("sit").setExecutor(new SitCommand(this));
    getCommand("lay").setExecutor(new LayCommand(this));
  }
  
  private void registerListeners()
  {
    PluginManager pm = Bukkit.getPluginManager();
    pm.registerEvents(new PlayerTeleport(), this);
    pm.registerEvents(new PlayerQuit(), this);
    pm.registerEvents(new PlayerArmorStandManipulate(), this);
    pm.registerEvents(new PlayerDeath(), this);
    pm.registerEvents(new PlayerJoin(this), this);
  }
  
  public Map<UUID, ArmorStand> getSeats()
  {
    return this.seats;
  }
  
  public Map<UUID, LayTask> getLaying()
  {
    return this.laying;
  }
  
  public boolean checkForUpdates()
  {
    return this.checkForUpdates;
  }
  
  public String getSitFailMessage()
  {
    return this.sitFailMessage;
  }
  
  public String getSitDownMessage()
  {
    return this.sitDownMessage;
  }
  
  public String getSitUpMessage()
  {
    return this.sitUpMessage;
  }
  
  public Permission getSitPermission()
  {
    return this.sitPermission;
  }
  
  public Permission getLayPermission()
  {
    return this.layPermission;
  }
  
  public String getLayDownMessage()
  {
    return this.layDownMessage;
  }
  
  public String getLayUpMessage()
  {
    return this.layUpMessage;
  }
  
  public String getLayFailMessage()
  {
    return this.layFailMessage;
  }
  
  public String getLayOverLapMessage()
  {
    return this.layOverlapMessage;
  }
  
  private void sendConfigError(String message, Level level)
  {
    getLogger().log(level, message);
  }
}


