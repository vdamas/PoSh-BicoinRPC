$hostname = "localhost"
$port = 61142
$username = ""
$password = ""

$cesc = New-Module -ScriptBlock {
    param(
      [Parameter(Mandatory=$true)]
      [ValidateNotNullOrEmpty()]
      $hostname,
      [Parameter(Mandatory=$true)]
      [ValidateNotNullOrEmpty()]
      $port,
      [Parameter(Mandatory=$true)]
      [ValidateNotNullOrEmpty()]
      $username,
      [Parameter(Mandatory=$true)]
      [ValidateNotNullOrEmpty()]
      $password
    )
    
    $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username,$password)))
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add('Authorization',("Basic {0}" -f $base64AuthInfo))
    $headers.Add('Accept','Application/Json')

    $id = 0
    function query{
        param(
         $method = 'getinfo',
         $params = @()
        )
        $global:id += 1  

        $request = @{ 'method'=$method; 'params'=@($params); 'id'=$global:id } | ConvertTo-Json
        try{
         $res = Invoke-WebRequest "http://localhost:$port" -Headers $headers -Body $request -Method Post
         (($res.Content | ConvertFrom-Json).result)
        }
        catch{
            Write-Host $_.Exception.Message -ForegroundColor Red
        }
    }

    export-modulemember -function query
} -AsCustomObject -ArgumentList @($hostname, $port, $username, $password)

    function FromUtcEpocTime ([long]$UnixTime)
    {
        $epoch = New-Object System.DateTime (1970, 1, 1, 0, 0, 0, [System.DateTimeKind]::Utc);
        return $epoch.AddSeconds($UnixTime);
    }

    function Get-BlockCount{
         $global:cesc.query("getblockcount")
    }

    function Get-Block{
         param(
          $height = $null
         )

         $global:cesc.query("getblock",$cesc.query("getblockhash", $height))
    }

    function Get-BlockTransactions{
         param(
          $blockheight = $null
         )

        $blockdata = Get-Block $blockheight
        $blocktx = @()
    
         foreach ($tx in $blockdata.tx){
          $rawtx = $cesc.query("getrawtransaction", $tx)     
          $decodedtx =$cesc.query("decoderawtransaction", $rawtx)
      
          $txid = $decodedtx.txid 
            
          $decodedtx.vout |%{
            $txdata = New-Object -TypeName psobject -Property @{
                txid = $txid
                type = 'vout'
                address =  $_.scriptPubKey.addresses[0]
                value = $_.value
            }
            $blocktx += $txdata 
          }

          $decodedtx.vin |%{
            $address = $_.address
            if ($_.address -ne $null) {
            
                $txdata = New-Object -TypeName psobject -Property @{
                    txid           = $txid
                    type       = "vin"
                    address        = $address
                    value          = $_.value
                }
            }else{
                $address = "coinbase" 
                $txdata = New-Object -TypeName psobject -Property @{
                    txid           = $txid
                    type       = "vin"
                    address        = $address
                    value          = ($blocktx | Measure-Object -Property value -Sum).Sum
                }
            }
            $blocktx += $txdata
                
           }
         } 
         $blocktx | sort txid, type 
     }
