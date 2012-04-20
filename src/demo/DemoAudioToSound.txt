package demo
{
	import tonfall.format.aiff.AIFFSound;
	import tonfall.format.pcm.PCMSound;
	import tonfall.format.wav.WAVSound;

	import flash.display.Sprite;
	import flash.events.Event;
	import flash.events.MouseEvent;
	import flash.net.FileFilter;
	import flash.net.FileReference;
	import flash.text.TextField;
	import flash.text.TextFieldAutoSize;
	import flash.text.TextFormat;

	/**
	 * Creates a flash.media.Sound by different audio decoders
	 * 
	 * @author Andre Michelle
	 */
	public final class DemoAudioToSound extends Sprite
	{
		private const fileRef : FileReference = new FileReference();
		private const textField : TextField = new TextField();
		
		public function DemoAudioToSound()
		{
			textField.autoSize = TextFieldAutoSize.LEFT;
			textField.defaultTextFormat = new TextFormat( 'Verdana', 10, 0x666666, true );
			textField.text = 'Click to browse from your local harddrive\n';
			addChild( textField );
		
			stage.addEventListener( MouseEvent.CLICK, click );
		}
		
		private function click( event : MouseEvent ) : void
		{
			stage.removeEventListener( MouseEvent.CLICK, click );
			
			textField.text = 'Browsing\n';
			
			browseAudioFile();
		}
		
		private function browseAudioFile() : void
		{
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
			
			var sound: PCMSound;
			
			try
			{
				switch( extension )
				{
					case 'wav':
						sound = new WAVSound( fileRef.data, onPCMSoundComplete );
						break;
						
					case 'aif':
					case 'aiff':
						sound = new AIFFSound( fileRef.data, onPCMSoundComplete );
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
			
			textField.appendText( 'Parsing Complete... waiting for flash.media.Sound.\n' );
		}

		private function onPCMSoundComplete( sound: PCMSound ): void
		{
			textField.appendText( 'seconds: ' + ( sound.length / 1000 ).toFixed(3) + '\n' );
			textField.appendText( 'playback... (reload to try again)\n' );
			
			sound.play();
		}
	}
}
