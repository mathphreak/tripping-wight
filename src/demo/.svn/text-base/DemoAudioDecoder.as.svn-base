package demo
{
	import tonfall.format.aiff.AIFFDecoder;
	import tonfall.format.pcm.PCMDecoder;
	import tonfall.format.wav.WAVDecoder;

	import flash.display.Sprite;
	import flash.events.Event;
	import flash.events.MouseEvent;
	import flash.events.SampleDataEvent;
	import flash.media.Sound;
	import flash.media.SoundChannel;
	import flash.net.FileFilter;
	import flash.net.FileReference;
	import flash.text.TextField;
	import flash.text.TextFieldAutoSize;
	import flash.text.TextFormat;
	import flash.utils.ByteArray;

	/**
	 * Showcasing different audio decoders
	 * 
	 * @author Andre Michelle
	 */
	[SWF(backgroundColor="#EDEDED", frameRate="31", width="512", height="192")]
	public final class DemoAudioDecoder extends Sprite
	{
		private static const BUFFER_SIZE : int = 2048;
		private const fileRef : FileReference = new FileReference();
		private const textField : TextField = new TextField();
		private const sound : Sound = new Sound();
		private const memory : ByteArray = new ByteArray();

		private var decoder : PCMDecoder;
		private var numSamples : Number;
		private var position : Number;
		private var soundChannel : SoundChannel;
		private var firstRun: Boolean;

		public function DemoAudioDecoder()
		{
			textField.autoSize = TextFieldAutoSize.LEFT;
			textField.defaultTextFormat = new TextFormat( 'Verdana', 10, 0x666666, true );
			textField.text = 'Click to browse from your local harddrive\n';
			addChild( textField );
			
			firstRun = true;
			
			stage.addEventListener( MouseEvent.CLICK, click );		
		}

		private function click( event : MouseEvent ) : void
		{
			stage.removeEventListener( MouseEvent.CLICK, click );
			
			if( firstRun )
			{
				firstRun = false;
			}
			else
			{
				textField.text = 'Browsing\n';
			}
			
			browseAudioFile();
		}

		private function browseAudioFile() : void
		{
			// allocate memory (sizeof float:4)*2
			memory.length = BUFFER_SIZE << 3;

			fileRef.addEventListener( Event.SELECT, select );
			fileRef.addEventListener( Event.CANCEL, cancel );
			fileRef.browse( [ new FileFilter( 'Wav', '*.wav' ), new FileFilter( 'Aiff', '*.aiff' ), new FileFilter( 'Aiff', '*.aif' ) ] );
		}

		private function cancel( event : Event ) : void
		{
			textField.appendText( 'Bye bye.\n' );
			
			fileRef.removeEventListener( Event.CANCEL, cancel );
			fileRef.removeEventListener( Event.SELECT, select );
		}

		private function select( event : Event ) : void
		{
			fileRef.removeEventListener( Event.CANCEL, cancel );
			fileRef.removeEventListener( Event.SELECT, select );

			fileRef.addEventListener( Event.COMPLETE, complete );
			fileRef.load();
		}

		private function complete( event : Event ) : void
		{
			fileRef.removeEventListener( Event.COMPLETE, complete );

			textField.appendText( 'Loaded ' + fileRef.name + ', fileSize: ' + fileRef.data.length + '\n' );
			
			var extension: String = fileRef.name;
			
			extension = extension.substr( extension.lastIndexOf( '.' ) + 1 );
			
			try
			{
				switch( extension )
				{
					case 'wav':
						decoder = new WAVDecoder( fileRef.data );
						break;
						
					case 'aif':
					case 'aiff':
						decoder = new AIFFDecoder( fileRef.data );
						break;
					
					default: throw new Error( 'Unknown extension.' );
				}
			}
			catch( e : Error )
			{
				textField.appendText( 'Error while parsing ' + e.name + ' occured.\n' );
				textField.appendText( e.message + '\n' );
				trace( e.getStackTrace() );
				return;
			}
			
			textField.appendText( '[Header] compression: ' + decoder.compressionType + '\n' );
			textField.appendText( '[Header] numChannels: ' + decoder.numChannels + '\n' );
			textField.appendText( '[Header] samplingRate: ' + decoder.samplingRate + '\n' );
			textField.appendText( '[Header] bits: ' + decoder.bits + '\n' );
			textField.appendText( '[Header] numSamples: ' + decoder.numSamples + '\n' );
			textField.appendText( '[Header] seconds: ' + decoder.seconds.toFixed( 3 ) + '\n' );
			textField.appendText( '[Header] supported: ' + decoder.supported + '\n' );
			
//			if ( 0 < decoder.ignoredTags.length )
//			{
//				// WAV format support additional information. Printing the IDs...
//				textField.appendText( '[Format] ignored tags: ' + decoder.ignoredTags + '\n' );
//			}

			if ( decoder.supported )
			{
				textField.appendText( 'playback...\n' );
				play();
			}
			else
			{
				finished();
			}
		}

		private function play() : void
		{
			numSamples = decoder.getNumSamples( 44100.0 );

			position = 0;

			sound.addEventListener( SampleDataEvent.SAMPLE_DATA, sampleData );
			soundChannel = sound.play();
		}

		private function sampleData( event : SampleDataEvent ) : void
		{
			const data : ByteArray = event.data;

			memory.position = 0;

			try
			{
				const read : Number = decoder.extract( memory, BUFFER_SIZE, position );
			}
			catch( e : Error )
			{
				textField.appendText( 'Error while parsing ' + e.name + 'occured.\n' );
				textField.appendText( e.message + '\n' );

				finished();
			}

			if ( 0 == read )
			{
				textField.appendText( 'complete.\n' );
				finished();
				return;
			}

			memory.position = 0;

			position += read;

			for ( var i : int = 0 ; i < read ; ++i )
			{
				var l: Number = memory.readFloat();
				var r: Number = memory.readFloat();
				
				data.writeFloat( l );
				data.writeFloat( r );
			}

			for ( ; i < BUFFER_SIZE ; ++i )
			{
				data.writeFloat( 0.0 );
				data.writeFloat( 0.0 );
			}
		}

		private function finished() : void
		{
			sound.removeEventListener( SampleDataEvent.SAMPLE_DATA, sampleData );
			
			textField.appendText( 'Click to do it again.\n' );
			
			stage.addEventListener( MouseEvent.CLICK, click );
		}
	}
}
