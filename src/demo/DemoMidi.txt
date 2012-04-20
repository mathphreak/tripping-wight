package demo
{
	import assets.Midi;

	import tonfall.core.TimeEventContainerSequencer;
	import tonfall.display.AbstractApplication;
	import tonfall.format.SoundDecoder;
	import tonfall.prefab.poly.PolySynth;

	import flash.events.Event;
	import flash.events.ProgressEvent;
	import flash.net.URLRequest;
	import flash.text.TextField;
	import flash.text.TextFormat;

	/**
	 * Demoes Midi
	 * 
	 * @author Andre Michelle
	 */
	[SWF(width='512',height='48',backgroundColor='0x1b1b1b',frameRate='32',scriptTimeLimit='255')]
	public final class DemoMidi extends AbstractApplication
	{
		private const sheet: SoundSheet = new SoundSheet();
		
		private const sequencer: TimeEventContainerSequencer = new TimeEventContainerSequencer( Midi.SCHUMANN.toTimeEventContainer( 1.0 / 128.0 ) );
		private const generator: PolySynth = new PolySynth( new SoundSheetVoiceFactory( sheet ) );

		private const textField: TextField = new TextField();

		private var _mp3: SoundDecoder;
		private var _inited: Boolean;

		public function DemoMidi()
		{
			textField.defaultTextFormat = new TextFormat( 'Verdana', 11, 0xFFCC00 );
			addChild( textField );
			
			loadSheet( '../load/piano.mp3' );
//			loadSheet( '../load/strings.mp3' );
//			loadSheet( '../load/rhodes.mp3' );
//			loadSheet( '../load/guitar.mp3' );
//			loadSheet( '../load/accordion.mp3' );
//			loadSheet( '../load/vibraphone.mp3' );
		}

		private function loadSheet( url: String ): void
		{
			generator.clear();
			driver.running = false;
			engine.barPosition = 0.0;
			
			if( null != _mp3 )
			{
				try { _mp3.close(); } catch( e: Error ) {}
				_mp3 = null;
			}

			_mp3 = new SoundDecoder( SoundDecoder.MP3_LAME_OFFSET );
			_mp3.addEventListener( Event.COMPLETE, mp3Complete );
			_mp3.addEventListener( ProgressEvent.PROGRESS, mp3Progress );
			_mp3.load( new URLRequest( url ) );
		}

		private function mp3Progress( event: ProgressEvent ): void
		{
			if( 0.0 < event.bytesTotal )
				textField.text = Math.ceil( ( event.bytesLoaded / event.bytesTotal * 100.0 ) ) + '%';
		}

		private function mp3Complete( event: Event ): void
		{
			textField.text = ( _mp3.bytesTotal >> 10 ) + 'kb';
			
			driver.running = true;
			
			_mp3.removeEventListener( Event.COMPLETE, mp3Complete );
			_mp3.removeEventListener( ProgressEvent.PROGRESS, mp3Progress );
			
			sheet.setDecoder( _mp3 );
			
			if( !_inited )
			{
				init();
			}
		}

		private function init(): void
		{
			sequencer.timeEventTarget = generator;
			engine.processors.push( sequencer );
			engine.processors.push( generator );
			engine.input = generator.signalOutput;
			
			_inited = true;
		}
	}
}
import tonfall.core.Engine;
import tonfall.core.Memory;
import tonfall.core.Signal;
import tonfall.core.TimeConversion;
import tonfall.core.TimeEvent;
import tonfall.core.TimeEventNote;
import tonfall.core.noteToFrequency;
import tonfall.format.IAudioDecoder;
import tonfall.prefab.poly.IPolySynthVoice;
import tonfall.prefab.poly.IPolySynthVoiceFactory;
import tonfall.util.ISoundSheet;

final class SoundSheet
	implements ISoundSheet
{
	public static const FREQUENCIES: Vector.<Number> = createFrequencyTable();

	private static function createFrequencyTable(): Vector.<Number>
	{
		const table: Vector.<Number> = new Vector.<Number>( 11, true );
		
		for( var i: int = 0 ; i < 11 ; ++i )
		{
			table[i] = noteToFrequency( 36 + i * 6 );
		}

		return table;
	}
	
	private var _decoder: IAudioDecoder;

	public function setDecoder( decoder: IAudioDecoder ): void
	{
		_decoder = decoder;
	}

	public function getNearestKeyIndexByNote( note: int ): int
	{
		var index: int = ( note - 36.0 ) / 6.0 + 0.5;

		if( index < 0 )
			index = 0;
		else
		if( index > 10 )
			index = 10;
			
		return index;
	}

	public function getFrequencyByKeyIndex( keyIndex: int ): Number
	{
		return FREQUENCIES[ keyIndex ];
	}

	public function get numSamplesEachKey(): Number
	{
		return 44100.0;
	}

	public function get decoder(): IAudioDecoder
	{
		return _decoder;
	}

	public function getStartPositionFromKeyIndex( keyIndex: int ): Number
	{
		return keyIndex * numSamplesEachKey;
	}

	public function getEndPositionFromKeyIndex( keyIndex: int ): Number
	{
		return ( keyIndex + 1 ) * numSamplesEachKey;
	}
}

final class SoundSheetVoiceFactory
	implements IPolySynthVoiceFactory
{
	private var _sheet: ISoundSheet;

	public function SoundSheetVoiceFactory( sheet: ISoundSheet )
	{
		_sheet = sheet;
	}
	
	public function create( event: TimeEventNote ): IPolySynthVoice
	{
		return new SoundSheetVoice( _sheet );
	}
}

final class SoundSheetVoice
	implements IPolySynthVoice
{
	// Lazy envelope built
	private const engine: Engine = Engine.getInstance();
	private const RELEASE_DURATION: int = 8820; // 200ms

	private var _rate: Number;
	private var _position: Number;
	private var _end: Number;
	private var _envelopePointer: int;
	private var _sheet: ISoundSheet;
	private var _velocity: Number;

	public function SoundSheetVoice( sheet: ISoundSheet )
	{
		_sheet = sheet;
	}
	
	public function start( event: TimeEvent ): void
	{
		const note: int = TimeEventNote( event ).note;

		const keyIndex: int = _sheet.getNearestKeyIndexByNote( note );
		
		const sampleFrequency: Number = _sheet.getFrequencyByKeyIndex( keyIndex );
		const noteFrequency: Number = noteToFrequency( note );

		_rate = noteFrequency / sampleFrequency;

		_position = _sheet.getStartPositionFromKeyIndex( keyIndex );
		_end = _sheet.getEndPositionFromKeyIndex( keyIndex );
		
		_velocity = TimeEventNote( event ).velocity;
		
		_envelopePointer = TimeConversion.barsToNumSamples( TimeEventNote( event ).barDuration, engine.bpm );
	}

	public function stop(): void
	{
	}

	public function processAdd( current: Signal, numSignals: int ): Boolean
	{
		//-- REUSE INSTEAD OF RECREATION
		Memory.position = 0;
		
		var p0: Number = _position;
		var p1: Number =  p0 + numSignals * _rate;
		
		if( p1 > _end )
			p1 = _end;

		const i0: int = p0;
		const i1: int = p1 + 2.0;
		
		//-- EXTRACT SAMPLES
		_sheet.decoder.extract( Memory, i1 - i0, i0 );
		
		var l0: Number;
		var r0: Number;
		var l1: Number;
		var r1: Number;
		
		var bufferPosition: Number = p0 - i0;
		
		var integer: int;
		var alpha: Number;
		var env: Number;

		for( var i: int = 0 ; i < numSignals ; ++i )
		{
			_position += _rate;

			if( _position >= _end )
				return true;
				
			if( 0 < _envelopePointer-- )
				env = 1.0;
			else
			if( -RELEASE_DURATION > _envelopePointer )
				return true;
			else
				env = 1.0 + _envelopePointer / RELEASE_DURATION;
			
			env *= _velocity;

			integer = bufferPosition;
			alpha = bufferPosition - integer;
			bufferPosition += _rate;

			//-- SET TARGET READ POSITION
			Memory.position = integer << 3;

			//-- READ TWO STEREO SAMPLES FOR LINEAR INTERPOLATION
			l0 = Memory.readFloat();
			r0 = Memory.readFloat();

			l1 = Memory.readFloat();
			r1 = Memory.readFloat();
			
			//-- WRITE INTERPOLATED AMPLITUDES INTO STREAM
			current.l += ( l0 + alpha * ( l1 - l0 ) ) * env;
			current.r += ( r0 + alpha * ( r1 - r0 ) ) * env;
			current = current.next;
		}
		
		return false;
	}

	public function dispose(): void
	{
		_sheet = null;
	}
}