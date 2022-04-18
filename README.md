# public
Public archives

Set-esxiHostFirewallRules.p1
[root@snow Set-esxi-HostFirewallRules]# cat Set-esxiHostFirewallRules.p1
Add-PSSnapin VMware.VimAutomation.Core
Add-PSSnapin SqlServerCmdletSnapin100
Add-PSSnapin SqlServerProviderSnapin100

# vCenter Server
$vc = "vcenter_fqdn"

# vCenter server credentials
$username = "username"
$password = "password"

# connect to the vCenter server with local account
Connect-VIServer $vc -User $username -Password $password

# Get list of clusters
$clusters = Get-Cluster

$iplist = @("192.168.0.0/24", "192.168.1.0/24")

#loop thru clusters
foreach($cluster in $clusters)
{
    $hostlist = Get-VMHost -Location $cluster
    foreach($esxihost in $hostlist)
    {
        # get esxicli for host
        $esxicli = Get-EsxCli -VMHost $esxihost.Name
        if($esxicli -ne $NULL)
        {
        # get list of firewall rules for the ruleset
        $rulelist = $esxicli.network.firewall.ruleset.list()



        #loop thru the rules
        foreach($rule in $rulelist | where {$_.enabled -ne "false" -and $_.name -ne "vmotion" -and $_.name -ne "vSphereClient"})
        {
            Write-Host("Changing allowed IP range for" + $servername + $rule.Name + "to include" + $iplist) -ForegroundColor Yellow
            $esxicli.network.firewall.ruleset.set($false, $true, $rule.Name)
            foreach($ip in $iplist)
            {
                $esxicli.network.firewall.ruleset.allowedip.add("$ip", $rule.Name)
            }
            # apply firewall rules
            $esxicli.network.firewall.refresh()
        }
        }
        $esxicli = $null

    }

}




Re-authored by Aaron Surina
  
