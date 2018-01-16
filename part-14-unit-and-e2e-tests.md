# Part 14 - Unit and e2e tests



    import { async, ComponentFixture, TestBed } from '@angular/core/testing';

    import { LoginFormComponent } from './login-form.component';
    import { MaterialModule } from '@demo-app/material';
    import { ReactiveFormsModule } from '@angular/forms';
    import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
    import { Authenticate } from '@demo-app/data-models';

    fdescribe('LoginFormComponent', () => {
      let component: LoginFormComponent;
      let fixture: ComponentFixture<LoginFormComponent>;

      beforeEach(
        async(() => {
          TestBed.configureTestingModule({
            imports: [MaterialModule, ReactiveFormsModule, BrowserAnimationsModule],
            declarations: [LoginFormComponent]
          }).compileComponents();
        })
      );

      beforeEach(() => {
        fixture = TestBed.createComponent(LoginFormComponent);
        component = fixture.componentInstance;
        fixture.detectChanges();
      });

      it('should create', () => {
        expect(component).toBeTruthy();
      });

      it(`should have a disabled form on start`, () => {
          expect(component.loginForm.invalid).toEqual(true);
        }); 

        it(`should be valid with authenticate object`, () => {
          const authenticate: Authenticate = {username: 'duncan', password: '123'};
          component.loginForm.setValue(authenticate);
          expect(component.loginForm.valid).toEqual(true);
      });
    });




