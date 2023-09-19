# HashiCorpVault

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install vault

cd /home/oracle/scripts/hashicorp/
nohup vault server -config=config.hcl &

export VAULT_ADDR='http://127.0.0.1:8200'
vault operator unseal OedYIBP6mp8moGKamVbovVK5cHcEVCApFdfqjz5XiC21
vault operator unseal Dn+7KkwIIF1Vn4y5p7mGOHNsVpeDKbdxoC91YrUVGj81


sudo useradd foo
sudo passwd foo


emctl set property -name "oracle.sysman.core.security.auth.cred_provider_prop1" -value "hvs.PW20y0sjdOhdz3Jx9riUAbvq"

#Creation of Cred Provider using HashiCorp
emcli config_cred_provider -provider_name="HashiVault" -provider_type=ScriptProvider -params="Command:sh %ScriptFile% -k %CredKey%;StdInVars:AuthToken;ScriptFileExt:sh;Script:file1" -input_file="file1:/home/oracle/scripts/hashicorp/scripts/getcreds.sh" -refparams="AuthToken:oracle.sysman.core.security.auth.cred_provider_prop1"

#Mapping Script output to EM Credential object
emcli store_cred_mapping -mapper_name="VaulttoEMHostCreds" -mapper_desc="Map Vault credential attributes to HostCreds type" -auth_target_type=host -cred_type="HostCreds" -attributesmap="pass:HostPassword"

emcli store_cred_mapping -mapper_name="VaulttoEMDBCreds" -mapper_desc="Map Vault credential attributes to DBCreds type" -auth_target_type=oracle_database -cred_type="DBCreds" -attributesmap="pass:DBPassword"

#Creation of PAM based named credentials
emcli create_named_credential -cred_name=VAULT_SYS -auth_target_type=oracle_database -cred_type=DBCreds -alt_cred_storage -cred_provider=HashiVault -cred_key="hrdb_sys;DBCreds" -cred_mapping=VaulttoEMDBCreds -attributes="DBUserName:sys;DBRole:SYSDBA"

emcli create_named_credential -cred_name=VAULT_FOO -auth_target_type=host -cred_type=HostCreds -target_type=host -cred_scope=global -alt_cred_storage -cred_provider=HashiVault -cred_key="hrhost_foo;DBCreds" -cred_mapping=VaulttoEMHostCreds -attributes="HostUserName:foo"
