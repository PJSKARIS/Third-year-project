unit gain;

interface

uses
  Winapi.Windows, Winapi.Messages, System.SysUtils, System.Variants, System.Classes, Vcl.Graphics,
  Vcl.Controls, Vcl.Forms, Vcl.Dialogs, Vcl.StdCtrls, VclTee.TeeGDIPlus,
  VCLTee.TeEngine, VCLTee.Series, Vcl.ExtCtrls, VCLTee.TeeProcs, VCLTee.Chart, mmsystem,
  Vcl.MPlayer, Math;

type
  TForm1 = class(TForm)
    Button1: TButton;
    OpenDialog1: TOpenDialog;
    Button2: TButton;
    Inputchart: TChart;
    Series1: TLineSeries;
    Outputchart: TChart;
    Series2: TLineSeries;
    Button3: TButton;
    Button4: TButton;
    highfreq: TEdit;
    lowfactor: TEdit;
    lowfreq: TEdit;
    highfactor: TEdit;
    Button5: TButton;
    Button6: TButton;
    Label3: TLabel;
    Label4: TLabel;
    Label5: TLabel;
    Label6: TLabel;
    Button7: TButton;
    highorder: TEdit;
    Label7: TLabel;
    midorder: TEdit;
    loworder: TEdit;
    Label1: TLabel;
    Label2: TLabel;
    Label8: TLabel;
    tsfunction: TChart;
    Series3: TLineSeries;
    Button8: TButton;
    Button13: TButton;
    Button14: TButton;
    Label11: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
    procedure Button3Click(Sender: TObject);
    procedure Button4Click(Sender: TObject);
    procedure Button5Click(Sender: TObject);
    procedure Button6Click(Sender: TObject);
    procedure Button7Click(Sender: TObject);
    procedure Button8Click(Sender: TObject);
    procedure Button9Click(Sender: TObject);
    procedure Button10Click(Sender: TObject);
    procedure Button11Click(Sender: TObject);
    procedure Button12Click(Sender: TObject);
    procedure Button13Click(Sender: TObject);
    procedure Button14Click(Sender: TObject);
  private
    { Private declarations }
  public
   Inputfile: string;
   Outputfile: string;


  end;

var
  Form1: TForm1;


implementation
type
    TWavheader = record
      idRIFF: array[0..3] of Ansichar;
      Chunksize: longint;
      idWave: array[0..3] of Ansichar;
      idFormat: array[0..3] of Ansichar;
      Subchunksieze: longint;
      Audiofmt: smallint;
      Numchan: smallint;
      Samplerate: longint;
      Byterate: longint;
      Allign: smallint;
      Bps: smallint;
    end;
   TDataheader = record
     idData: array[0..3] of Ansichar;
     datasize: longint;
   end;

{$R *.dfm}






procedure TForm1.Button1Click(Sender: TObject);


var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  datafactorout: smallint;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  threshold1: single;
  threshold2: single;
  a : real;

begin

        Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        threshold1 := StrToFloat(highfreq.Text);
        threshold2 := StrToFloat(lowfreq.Text);
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
         myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;
        if numofsamp > 0 then
        for i := 0 to numofsamp -1 do
          begin
            Sample := datain[i];
            datafactorin := Sample/32767;
            if datafactorin > threshold1 then datafactorin := threshold1;
            if datafactorin < threshold2 then datafactorin := threshold2;
            datafactorout := trunc(datafactorin * 32767);
            dataout[i] := datafactorout;
          end;
        for i := 0 to numofsamp -1 do
           begin
             Sample := dataout[i];
             a := Sample/32767;
             Outputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;


        Outputfile := 'D:\output.wav';
         if numofsamp > 0 then
         myFile := TFileStream.Create(Outputfile, fmCreate);
         try
         myFile.Writebuffer(Waveheader, sizeof(TWavheader));
         myFile.Writebuffer(Dataheader, sizeof(TDataheader));
         myFile.Writebuffer(dataout[0], Dataheader.datasize);
         finally
         myFile.free;
         end;

end;

procedure TForm1.Button2Click(Sender: TObject);
begin
 sndPlaySound(PChar(Inputfile), SND_ASYNC or SND_FILENAME);

end;

procedure TForm1.Button3Click(Sender: TObject);
begin

 sndPlaySound(PChar(Outputfile), SND_ASYNC or SND_FILENAME);

end;

procedure TForm1.Button4Click(Sender: TObject);


var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  datafactorout: smallint;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  lowlimit, highlimit: real;
  factorhigh, factorlow : single;
  a : real;
begin
        Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        lowlimit := StrtoFloat(lowfreq.text);
        highlimit := StrToFloat(highfreq.text);
        factorlow := StrToFloat(lowfactor.text);
        factorhigh := StrToFloat(highfactor.text);
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
            myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;
        if numofsamp > 0 then
        for i := 0 to numofsamp -1 do
          begin
            Sample := datain[i];
            datafactorin := Sample/32767;
            if datafactorin > 0 then (if datafactorin > highlimit then datafactorin := datafactorin * factorhigh);
            if datafactorin > 0 then (if datafactorin < lowlimit then datafactorin := datafactorin * factorlow);
            if datafactorin < 0 then (if datafactorin > (-1 * lowlimit) then datafactorin := datafactorin *factorlow);
            if datafactorin < 0 then (if datafactorin < (-1 * highlimit) then datafactorin := datafactorin *factorhigh);
            if datafactorin > 1 then datafactorin := 1;
            if datafactorin < -1 then datafactorin := -1;
            datafactorout := trunc(datafactorin * 32767);
            dataout[i] := datafactorout;
          end;
        for i := 0 to numofsamp -1 do
           begin
             Sample := dataout[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Outputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;


        Outputfile := 'D:\output.wav';
         if numofsamp > 0 then
         myFile := TFileStream.Create(Outputfile, fmCreate);
         try
         myFile.Writebuffer(Waveheader, sizeof(TWavheader));
         myFile.Writebuffer(Dataheader, sizeof(TDataheader));
         myFile.Writebuffer(dataout[0], Dataheader.datasize);
         finally
         myFile.free;
         end;

end;

procedure TForm1.Button5Click(Sender: TObject);
var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  threshold1: single;
  threshold2: single;
  a : real;

begin
Opendialog1.execute();
        Inputfile := Opendialog1.filename;
        Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
         myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;



end;

procedure TForm1.Button6Click(Sender: TObject);
begin
sndPlaySound(nil, SND_PURGE);
end;



procedure TForm1.Button7Click(Sender: TObject);
var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  datafactorout: smallint;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  lowlimit, highlimit, uporder, order, downorder: real;
  factorhigh, factorlow : single;
  a : real;
begin
        Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        order := StrToFloat(midorder.text);
        downorder := StrToFloat(loworder.text);
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
            myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;
        if numofsamp > 0 then
        for i := 0 to numofsamp -1 do
          begin
            Sample := datain[i];
            datafactorin := Sample/32767;
            datafactorin := (uporder*datafactorin-order*power(datafactorin,3)+downorder*power(datafactorin,5));
            if datafactorin > 1 then datafactorin := 1;
            if datafactorin < -1 then datafactorin := -1;
            datafactorout := trunc(datafactorin * 32767);
            dataout[i] := datafactorout;
          end;
        for i := 0 to numofsamp -1 do
           begin
             Sample := dataout[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Outputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;


        Outputfile := 'D:\output.wav';
         if numofsamp > 0 then
         myFile := TFileStream.Create(Outputfile, fmCreate);
         try
         myFile.Writebuffer(Waveheader, sizeof(TWavheader));
         myFile.Writebuffer(Dataheader, sizeof(TDataheader));
         myFile.Writebuffer(dataout[0], Dataheader.datasize);
         finally
         myFile.free;
         end;
end;



procedure TForm1.Button8Click(Sender: TObject);
var
 uporder, order, downorder, x, y: real;
begin
        Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        tsfunction.bottomaxis.title.caption := 'Input';
        tsfunction.leftaxis.title.caption := 'Output';
        tsfunction.title.caption := 'Transfer function';
        tsfunction.series[0].clear;
        tsfunction.leftaxis.maximumoffset := 30;
        tsfunction.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        order := StrToFloat(midorder.text);
        downorder := StrToFloat(loworder.text);
        x := -1;
        while x <= 1 do
        begin
        y := uporder + order * x + downorder * power(x,2);
        tsfunction.series[0].addxy(x,y, '', clblue);
        x := x + 0.0001;
        end;


end;

procedure TForm1.Button10Click(Sender: TObject);
var
 uporder, order, downorder, x, y: real;
begin
       Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        tsfunction.bottomaxis.title.caption := 'Input';
        tsfunction.leftaxis.title.caption := 'Output';
        tsfunction.title.caption := 'Transfer function';
        tsfunction.series[0].clear;
        tsfunction.leftaxis.maximumoffset := 30;
        tsfunction.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        x := -1;
        while x <= 1 do
        begin
        y := x + uporder * power(x,3);
        tsfunction.series[0].addxy(x,y, '', clblue);
        x := x + 0.0001;
        end;
end;

procedure TForm1.Button12Click(Sender: TObject);
var
 uporder, order, downorder, x, y: real;
begin
       Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        tsfunction.bottomaxis.title.caption := 'Input';
        tsfunction.leftaxis.title.caption := 'Output';
        tsfunction.title.caption := 'Transfer function';
        tsfunction.series[0].clear;
        tsfunction.leftaxis.maximumoffset := 30;
        tsfunction.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        order := StrToFloat(midorder.text);
        x := -1;
        while x <= 1 do
        begin
        y := uporder * sin(x*order);
        tsfunction.series[0].addxy(x,y, '', clblue);
        x := x + 0.0001;
        end;
end;

procedure TForm1.Button14Click(Sender: TObject);
var
 uporder, order, downorder, x, y: real;
begin
     Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        tsfunction.bottomaxis.title.caption := 'Input';
        tsfunction.leftaxis.title.caption := 'Output';
        tsfunction.title.caption := 'Transfer function';
        tsfunction.series[0].clear;
        tsfunction.leftaxis.maximumoffset := 30;
        tsfunction.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        order := StrToFloat(midorder.text);
        downorder := StrToFloat(loworder.text);
        x := -1;
        while x <= 1 do
        begin
        y := uporder * tanh(order*power(x,downorder));
        tsfunction.series[0].addxy(x,y, '', clblue);
        x := x + 0.0001;
        end;
end;


procedure TForm1.Button9Click(Sender: TObject);
var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  datafactorout: smallint;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  lowlimit, highlimit: real;
  factorhigh, factorlow : single;
  a : real;
  uporder : real;
begin
       Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
            myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;
        if numofsamp > 0 then
        for i := 0 to numofsamp -1 do
          begin
            Sample := datain[i];
            datafactorin := Sample/32767;
            datafactorin := (datafactorin + uporder*power(datafactorin,3));
            if datafactorin > 1 then datafactorin := 1;
            if datafactorin < -1 then datafactorin := -1;
            datafactorout := trunc(datafactorin * 32767);
            dataout[i] := datafactorout;
          end;
        for i := 0 to numofsamp -1 do
           begin
             Sample := dataout[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Outputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;


        Outputfile := 'D:\output.wav';
         if numofsamp > 0 then
         myFile := TFileStream.Create(Outputfile, fmCreate);
         try
         myFile.Writebuffer(Waveheader, sizeof(TWavheader));
         myFile.Writebuffer(Dataheader, sizeof(TDataheader));
         myFile.Writebuffer(dataout[0], Dataheader.datasize);
         finally
         myFile.free;
         end;
end;

procedure TForm1.Button11Click(Sender: TObject);
var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  datafactorout: smallint;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  lowlimit, highlimit: real;
  factorhigh, factorlow : single;
  a : real;
  uporder, order: real;
begin
      Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        order := StrToFloat(midorder.text);
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
            myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;
        if numofsamp > 0 then
        for i := 0 to numofsamp -1 do
          begin
            Sample := datain[i];
            datafactorin := Sample/32767;
            datafactorin := (uporder * sin(datafactorin*order));
            if datafactorin > 1 then datafactorin := 1;
            if datafactorin < -1 then datafactorin := -1;
            datafactorout := trunc(datafactorin * 32767);
            dataout[i] := datafactorout;
          end;
        for i := 0 to numofsamp -1 do
           begin
             Sample := dataout[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Outputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;


        Outputfile := 'D:\output.wav';
         if numofsamp > 0 then
         myFile := TFileStream.Create(Outputfile, fmCreate);
         try
         myFile.Writebuffer(Waveheader, sizeof(TWavheader));
         myFile.Writebuffer(Dataheader, sizeof(TDataheader));
         myFile.Writebuffer(dataout[0], Dataheader.datasize);
         finally
         myFile.free;
         end;
end;

procedure TForm1.Button13Click(Sender: TObject);
var
  myFile: TFilestream;
  Waveheader: TWavheader;
  Dataheader: TDataheader;
  datain, dataout: array of smallint;
  datafactorin: single;
  datafactorout: smallint;
  Sample: single;
  i: integer;
  numofsamp, frequency: integer;
  channel, bitpersamp: Smallint;
  lowlimit, highlimit: real;
  factorhigh, factorlow : single;
  a : real;
  uporder, order, downorder: real;
begin
      Inputchart.Title.caption := 'Input';
        Inputchart.Bottomaxis.title.caption := 'Time';
        Outputchart.Title.caption := 'Output';
        Outputchart.Bottomaxis.title.caption := 'Time';
        Inputchart.series[0].clear;
        Outputchart.series[0].clear;
        Inputchart.leftaxis.maximumoffset := 30;
        Inputchart.leftaxis.minimumoffset := 30;
        Outputchart.leftaxis.maximumoffset := 30;
        Outputchart.leftaxis.minimumoffset := 30;
        uporder := StrtoFloat(highorder.text);
        order := StrToFloat(midorder.text);
        downorder := StrToFloat(loworder.text);
        myFile := TFileStream.Create(inputfile, fmOpenRead);
        try
        myFile.readbuffer(Waveheader, sizeof(TWavheader));
        if (Waveheader.idRIFF <> 'RIFF') or (Waveheader.idWave <> 'WAVE') then
         raise Exception.Create('Selected file is not a valid WAV file.');
        bitpersamp := Waveheader.Bps;
        channel := Waveheader.Numchan;
        frequency := Waveheader.Samplerate;
        if (bitpersamp = 0) or (channel = 0) then
        raise Exception.Create('Invalid WAV header: bitspersamp or channel is zero.');
        myFile.readbuffer(Dataheader, sizeof(TDataheader));
        numofsamp := Dataheader.datasize div (channel * (bitpersamp div 8));
        Setlength(datain, numofsamp);
        SetLength(dataout, numofsamp);
        myFile.Readbuffer(datain[0], Dataheader.datasize);
        finally
            myFile.free;
        end;

         if numofsamp > 0 then
         for i := 0 to numofsamp -1 do
           begin
             Sample := datain[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Inputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;
        if numofsamp > 0 then
        for i := 0 to numofsamp -1 do
          begin
            Sample := datain[i];
            datafactorin := Sample/32767;
            datafactorin := (uporder * tanh(order*power(datafactorin, downorder)));
            if datafactorin > 1 then datafactorin := 1;
            if datafactorin < -1 then datafactorin := -1;
            datafactorout := trunc(datafactorin * 32767);
            dataout[i] := datafactorout;
          end;
        for i := 0 to numofsamp -1 do
           begin
             Sample := dataout[i];
             if Sample > 35000 then Sample := 35000;
             if Sample < -35001 then Sample := -35001;
             a := Sample/32767;
             Outputchart.series[0].addxy(i/frequency, a, '', clblue);
           end;


        Outputfile := 'D:\output.wav';
         if numofsamp > 0 then
         myFile := TFileStream.Create(Outputfile, fmCreate);
         try
         myFile.Writebuffer(Waveheader, sizeof(TWavheader));
         myFile.Writebuffer(Dataheader, sizeof(TDataheader));
         myFile.Writebuffer(dataout[0], Dataheader.datasize);
         finally
         myFile.free;
         end;
end;





end.
