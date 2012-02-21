#!/usr/bin/perl
#
#  Juan Pedro Paredes <juampe@iquis.com>
#
#  PMQ Print MQSeries status
#
#  This program is free software; you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 2 of the License, or
#  (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with this program; if not, write to the Free Software
#  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
#

use Getopt::Long;

%mqchstat=();
%mqchstatchg=();
sub chstatus{
        #TODO Change this path to exec mqcommand
        my $mqstatus=`/opt/mqm/bin/mqcommand "display chstatus (*) all"`;
        #my @chans=$mqstatus=~/CHANNEL\([\w\.]*\).*?RQMNAME\([\w.]*\)/gsm;
        my @chans=$mqstatus=~/CHANNEL\([\w\.]*\).*?STOPREQ\([\w.]*\)/gsm;
        #@schans=sort(@chans);
        print
"CHANNEL              TYPE SPEED  STATUS  DBT  CURSEQNO LSTMSGTI CHG\n";
        print
"--------------------------------------------------------------------------------\n";
        foreach my $chan (@chans){
                my ($channel)=$chan=~/CHANNEL\(([\w\.]*)\)/gsm;
                my ($status)=$chan=~/\sSTATUS\((\w*)\)/gsm;
                my ($indoubt)=$chan=~/\sINDOUBT\((\w*)\)/gsm;
                my ($npmspeed)=$chan=~/\sNPMSPEED\((\w*)\)/gsm;
                my ($curseqno)=$chan=~/\sCURSEQNO\((\w*)\)/gsm;
                my ($chltype)=$chan=~/\sCHLTYPE\((\w*)\)/gsm;
                my ($lstmsgti)=$chan=~/\sLSTMSGTI\(([\w\.]*)\)/gsm;
                my $key="$channel,$chltype,$npmspeed,$status,$indoubt,$curseqno,$msgs,$lstmsgti";
                my $mqchange=".";
                if(defined($mqchstat{$channel}) and ($mqchstat{$channel} ne $key)){
                        $mqchange="X";
                        $mqchstatchg{$channel}=$mqchstat{$channel};
                }elsif(defined($mqchstatchg{$channel})){
                        $mqchange="-";
                }
                printf "%-20.20s %-4s %-6.6s %-7.7s %-3.3s %9s %8s  %1s\n",$channel,$chltype,$npmspeed,$status,$indoubt,$curseqno,$lstmsgti,$mqchange;

                $mqchstat{$channel}=$key;
        }
}

%mqqstat=();
%mqqstatchg=();
sub qstatus{
        my $mqstatus=`/opt/mqm/bin/mqcommand "display qlocal(*) all"`;
        my @queues=$mqstatus=~/DESCR\([\w\.\s]*\).*?CURDEPTH\([\w.]*\)/gsm;
        @squeues=sort(@queues);
        print
"QUEUE                                    GET PUT  MAXDEPTH  CURDEPTH CHG\n";
        print
"--------------------------------------------------------------------------------\n";
        foreach my $q (@queues){
                my ($queue)=$q=~/\sQUEUE\(([\w\.]*)\)/gsm;
                my ($get)=$q=~/\sGET\((\w*)\)/gsm;
                my ($put)=$q=~/\sPUT\((\w*)\)/gsm;
                my ($maxdepth)=$q=~/\sMAXDEPTH\((\w*)\)/gsm;
                my ($curdepth)=$q=~/\sCURDEPTH\((\w*)\)/gsm;
                my $key="$queue,$get,$put,$maxdepth,$curdepth";
                my $mqchange=".";
                if(defined($mqqstat{$queue}) and ($mqqstat{$queue} ne $key)){
                        $mqchange="X";
                        $mqqstatchg{$queue}=$mqqstat{$queue};
                }elsif(defined($mqqstatchg{$queue})){
                        $mqchange="-";
                }
                if(($curdepth>0) or (defined($mqqstatchg{$queue}))){
                        printf "%-40.40s %-3.3s %-3.3s %9s %9s  %1s\n",$queue,$get,$put,$maxdepth,$curdepth,$mqchange;
                }

                $mqqstat{$queue}=$key;
        }
}

sub report{
        my $mqstatus=`/opt/mqm/bin/mqcommand "display chstatus (*) all"`;
        my @chans=$mqstatus=~/CHANNEL\([\w\.]*\).*?STOPREQ\([\w.]*\)/gsm;
        foreach my $chan (@chans){
                my ($channel)=$chan=~/CHANNEL\(([\w\.]*)\)/gsm;
                my ($status)=$chan=~/\sSTATUS\((\w*)\)/gsm;
                my ($curseqno)=$chan=~/\sCURSEQNO\((\w*)\)/gsm;
                print "C,$channel,$status,$curseqno\n";
        }

        my $mqstatus=`/opt/mqm/bin/mqcommand "display qlocal(*) all"`;
        my @queues=$mqstatus=~/DESCR\([\w\.\s]*\).*?CURDEPTH\([\w.]*\)/gsm;
        @squeues=sort(@queues);
        foreach my $q (@queues){
                my ($queue)=$q=~/\sQUEUE\(([\w\.]*)\)/gsm;
                my ($maxdepth)=$q=~/\sMAXDEPTH\((\w*)\)/gsm;
                my ($curdepth)=$q=~/\sCURDEPTH\((\w*)\)/gsm;
                printf "Q,$queue,$maxdepth,$curdepth\n";
        }
}



$help='';
$channel='';
$queue='';
$report='';
$times='1';
$wait='5';
$result = GetOptions ("help|h"=>\$help,"channel|c"=> \$channel,"queue|q"=> \$queue,"report|r"=> \$report,"times|t=i"=>\$times,"wait|w=i"=>\$wait) or die "$me: bad options\n";

if(($times lt -1) or ($wait lt 5)or(($channel eq '')and($queue eq '')and($report eq ''))){
        $help=1;
}

if($help){
        print <<"EOF";
pmq Juan Pedro Paredes Caballero <juampe\@iquis.com>
Muestra informacion de Objetos de MQSeries.
Si no se especifica nada muesta el estado de los canales.
Uso: pmq [OPCION]...
  -h, --help    Informa del uso de pmq
  -c, --channel Informa del estado de las canales (por defecto)
  -q, --queue   Informa del estado de las colas con mensajes
  -r, --report  Informe para monitorizacion
  -t, --times   Muestra la inforacion x veces (1 por defecto, -1 siempre)
  -w, --wait    Espera x segundos para refrescar (5 por defecto o mayor que 5)
EOF
        exit 0;
}

for(my $i=$times;($times eq -1)or($i > 0);$i--){
        if($report eq 1){
                report();
                exit 0;
        }
        system "clear";
        if($channel eq 1){
                chstatus();
        }
        if($queue eq 1){
                qstatus();
        }
        if($report eq 1){
                report();
        }
        if(($times eq -1)or($i > 1)){
                sleep $wait;
        }
}

